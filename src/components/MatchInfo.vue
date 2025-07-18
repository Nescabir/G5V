<template>
  <v-container class="mapstatsinfo" fluid>
    <v-flex class="text-right" v-if="AdminToolsAvailable(matchInfo)">
      <AdminButton
        :matchInfo="matchInfo"
        :user="user"
        @force-the-reload="getMatchInfo()"
      />
    </v-flex>
    <v-flex v-else />
    <div class="text-h4 names" align="center">
      <router-link
        v-if="matchInfo.team1.id != 0"
        :to="{ path: '/teams/' + matchInfo.team1_id }"
      >
        <div v-if="matchInfo.team1.logo != null">
          <img
            :src="apiUrl + '/static/img/logos/' + matchInfo.team1.logo + '.png'"
            style="border-radius: 5px; width: 40px; height: 32px;"
            @error="imgUrlAlt"
          />
          {{ matchInfo.team1_name }}
        </div>
        <div v-else-if="matchInfo.team1.flag != null">
          <img
            :src="get_flag_link(matchInfo.team1)"
            style="border-radius: 5px;"
          />
          {{ matchInfo.team1_name }}
        </div>
        <div v-else>
          {{ matchInfo.team1_name }}
        </div>
      </router-link>
      <div v-else>
        {{ matchInfo.team1_name }}
      </div>
      {{ $t("Matches.Versus") }}
      <router-link
        v-if="matchInfo.team2.id != 0"
        :to="{ path: '/teams/' + matchInfo.team2_id }"
      >
        <div v-if="matchInfo.team2.logo != null">
          <img
            :src="apiUrl + '/static/img/logos/' + matchInfo.team2.logo + '.png'"
            style="border-radius: 5px; width: 40px; height: 32px;"
            @error="imgUrlAlt"
          />
          {{ matchInfo.team2_name }}
        </div>
        <div v-else-if="matchInfo.team2.flag != null">
          <img
            :src="get_flag_link(matchInfo.team2)"
            style="border-radius: 5px;"
          />
          {{ matchInfo.team2_name }}
        </div>
        <div v-else>
          {{ matchInfo.team2_name }}
        </div>
      </router-link>
      <div v-else>
        {{ matchInfo.team2_name }}
      </div>
    </div>
    <div class="final-score text-h4" align="center">
      {{ matchInfo.team1_score }} {{ matchInfo.symbol }}
      {{ matchInfo.team2_score }}
    </div>
    <div class="start-date text-subtitle-2" align="center">
      {{ $t("Match.StartTime") }} {{ matchInfo.start_time }}
    </div>
    <div
      class="end-date text-subtitle-2"
      align="center"
      v-if="matchInfo.end_time != null"
    >
      {{ $t("Match.EndTime") }} {{ matchInfo.end_time }}
    </div>
    <div v-if="matchInfo.forfeit == 1" align="center">
      <strong>
        {{ $t("Match.MatchForfeitedBy", { team: get_loser(matchInfo) }) }}
      </strong>
    </div>
    <div v-else-if="matchInfo.cancelled == 1" align="center">
      <strong>
        {{ $t("Match.MatchHasBeenCancelled") }}
      </strong>
    </div>
    <div
      align="center"
      v-if="
        user.id != null &&
          serverInfo.ip_string != '' &&
          matchInfo.end_time == null
      "
    >
      <v-btn
        color="primary"
        small
        :href="
          'steam://rungame/730/' +
            user.steam_id +
            '/+connect%20' +
            serverInfo.ip_string +
            ':' +
            serverInfo.port
        "
      >
        {{ $t("Match.Connect") }}
      </v-btn>
      <div v-if="serverInfo.gotv_port != null">
        <v-btn
          color="secondary"
          small
          :href="
            'steam://rungame/730/' +
              user.steam_id +
              '/+connect%20' +
              serverInfo.ip_string +
              ':' +
              serverInfo.gotv_port
          "
        >
          {{ $t("Match.GOTVConnect") }}
        </v-btn>
      </div>
    </div>
  </v-container>
</template>

<script>
import AdminButton from "./MatchAdminButton";
export default {
  components: {
    AdminButton
  },
  props: {
    match_id: Number,
    user: Object
  },
  sse: {
    cleanup: true
  },
  data() {
    return {
      matchInfo: {
        team1_name: "",
        team2_name: "",
        match_title: "",
        start_time: "",
        end_time: "",
        team1_id: 0,
        team2_id: 0,
        team1_score: 0,
        team2_score: 0,
        team1: {
          id: 0,
          user_id: 0,
          name: "",
          tag: "",
          flag: "",
          logo: "",
          auth_name: {},
          public_team: false
        },
        team2: {
          id: 0,
          user_id: 0,
          name: "",
          tag: "",
          flag: "",
          logo: "",
          auth_name: {},
          public_team: false
        },
        symbol: "",
        cancelled: 0,
        forfeit: 0,
        id: -1,
        user_id: -1
      },
      serverInfo: {
        ip_string: "",
        port: 0,
        gotv_port: 0
      },
      apiUrl: process.env?.VUE_APP_G5V_API_URL || "/api",
      imageLoaded: true
    };
  },
  created() {
    this.checkIfMatchLive();
  },
  methods: {
    async checkIfMatchLive() {
      let matchRes = await this.GetMatchData(this.match_id);
      if (matchRes.end_time == null) 
      {
        //await this.getStreamedMatchInfo();
        await this.getMatchInfo();
      }
      else await this.getMatchInfo();
    },
    async getStreamedMatchInfo() {
      try {
        let sseClient = await this.GetEventMatchData(this.match_id);
        await sseClient.connect();
        await sseClient.on("matches", async message => {
          try {
            await this.retrieveMatchInfoHelper(message);
          } catch (error) {
            console.error(
              "Error retrieving information from matches event stream. ",
              error
            );
          }
        });
        return;
      } catch (ignored) {
        return;
      }
    },
    async getMatchInfo() {
      try {
        let matchRes = await this.GetMatchData(this.match_id);
        await this.retrieveMatchInfoHelper(matchRes);
      } catch (error) {
        console.log(error);
      }
    },
    async retrieveMatchInfoHelper(serverResponse) {
      try {
        let team1Res = await this.GetBasicTeamInfo(serverResponse.team1_id);
        let team2Res = await this.GetBasicTeamInfo(serverResponse.team2_id);
        let serveRes = await this.GetServerData(serverResponse.server_id);
        this.matchInfo.team1_name = serverResponse.team1_string;
        this.matchInfo.team2_name = serverResponse.team2_string;
        this.matchInfo.team1_id = serverResponse.team1_id;
        this.matchInfo.team2_id = serverResponse.team2_id;
        this.matchInfo.start_time = new Date(
          serverResponse.start_time
        ).toLocaleString();
        this.matchInfo.end_time =
          serverResponse.end_time == null
            ? null
            : new Date(serverResponse.end_time).toLocaleString();
        this.matchInfo.team1_score = serverResponse.team1_score;
        this.matchInfo.team2_score = serverResponse.team2_score;
        this.matchInfo.symbol = this.GetScoreSymbol(
          this.matchInfo.team1_score,
          this.matchInfo.team2_score
        );
        this.matchInfo.team1 = team1Res;
        this.matchInfo.team2 = team2Res;
        this.matchInfo.cancelled = serverResponse.cancelled;
        this.matchInfo.forfeit = serverResponse.forfeit;
        this.matchInfo.id = this.match_id;
        this.matchInfo.user_id = serverResponse.user_id;
        if (serveRes) {
          this.serverInfo.ip_string = serveRes.ip_string;
          this.serverInfo.port = serveRes.port;
          this.serverInfo.gotv_port = serveRes.gotv_port;
        }
      } catch (err) {
        console.log(`Error on match helper. The error is ${err.toString()}`);
      }
    },
    imgUrlAlt(event) {
      if (event.target.src.includes("svg")) this.imageLoaded = false;
      else event.target.src = event.target.src.replace("png", "svg");
    }
  }
};
</script>
