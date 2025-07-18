<template>
  <v-container class="statistics" fluid>
    <div v-if="playerstats.length > 0">
      <v-container
        v-for="(playerMapStats, index) in playerstats"
        :key="playerMapStats[0].id"
      >
        <v-container class="mapinfo" fluid>
          <div
            class="text-subtitle-2 mapInfo"
            v-if="mapStats[index] != null"
            align="center"
          >
            {{ mapStats[index].score }} - {{ mapStats[index].map }}
          </div>
          <div
            class="text-subtitle-2 mapInfo"
            v-if="
              mapStats[index] != null && mapStats[index].start != null
            "
            align="center"
          >
            {{ mapStats[index].start }}
          </div>
          <div
            class="text-subtitle-2 mapInfo"
            v-if="
              mapStats[index] != null && mapStats[index].end != null
            "
            align="center"
          >
            {{ mapStats[index].end }}
          </div>
          <div
            class="text-subtitle-2 mapInfo"
            v-if="
              mapStats[index] != null && mapStats[index].demo != null
            "
            align="center"
          >
            <v-btn
              small
              color="secondary"
              :href="apiUrl + '/demo/' + mapStats[index].demo"
            >
              {{ $t("PlayerStats.Download") }}
            </v-btn>
          </div>
          <div
            class="text-subtitle-2 mapInfo"
            v-if="
              mapStats[index] != null && mapStats[index].end == null
            "
            align="left"
          ></div>
        </v-container>
        <v-data-table
          item-key="id"
          class="elevation-1"
          :items-per-page="12"
          :loading="isLoading"
          :loading-text="$t('misc.LoadText')"
          :headers="headers"
          :items="playerMapStats"
          sort-by="kills"
          sort-desc
          ref="PlayerStats"
          group-by="Team"
          show-group-by
          hide-default-footer
          show-expand
        >
          <template v-slot:item.name="{ item }">
            <router-link :to="{ path: '/user/' + item.steam_id }">
              {{ item.name }}
            </router-link>
          </template>
          <template v-slot:item.Team="{ item }">
            {{ item.Team.slice(2) }}
          </template>
          <template v-slot:expanded-item="{ item }" class="text-center">
            <td :colspan="headers.length">
              <v-data-table
                item-key="id"
                class="elevation-1"
                :headers="additionalHeaders"
                hide-default-footer
                dense
                :key="item.id"
                :items="[item]"
                disable-sort
                :colspan="headers.length"
              />
            </td>
          </template>
        </v-data-table>
      </v-container>
    </div>
    <div v-else align="center">
      <strong>
        {{ $t("PlayerStats.NoStatFound") }}
      </strong>
    </div>
  </v-container>
</template>

<script>
export default {
  props: {
    match_id: Number
  },
  data() {
    return {
      playerstats: [],
      isLoading: true,
      mapStats: [],
      allowRefresh: false,
      timeoutId: -1,
      isFinished: false,
      apiUrl: process.env?.VUE_APP_G5V_API_URL || "/api"
    };
  },
  created() {
    this.useStreamOrStaticData();
  },
  computed: {
    headers() {
      return [
        {
          text: this.$t("PlayerStats.User"),
          align: "start",
          sortable: true,
          value: "name",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.Kills"),
          value: "kills",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.Deaths"),
          value: "deaths",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.Assists"),
          value: "assists",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.FlashbangAssists"),
          value: "flashbang_assists",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.EnemyFlash"),
          value: "enemies_flashed",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.FriendFlash"),
          value: "friendlies_flashed",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.Rating"),
          value: "rating",
          groupable: false
        },
        {
          text: this.$t("PlayerStats.TeamName"),
          value: "Team",
          align: "right"
        },
        {
          text: "",
          value: "data-table-expand",
          groupable: false,
          align: "end"
        }
      ];
    },
    additionalHeaders() {
      return [
        {
          text: this.$t("PlayerStats.Suicides"),
          value: "suicides"
        },
        {
          text: this.$t("PlayerStats.ADR"),
          value: "adr"
        },
        {
          text: this.$t("PlayerStats.UtilDamage"),
          value: "util_damage"
        },
        {
          text: this.$t("PlayerStats.KnifeKills"),
          value: "knife_kills"
        },
        {
          text: this.$t("PlayerStats.BombPlants"),
          value: "bomb_plants"
        },
        {
          text: this.$t("PlayerStats.BombDefuses"),
          value: "bomb_defuses"
        },
        {
          text: this.$t("PlayerStats.Headshot") + "%",
          value: "hsp"
        },
        {
          text: this.$t("PlayerStats.KDR"),
          value: "kdr"
        },
        {
          text: this.$t("PlayerStats.FPR"),
          value: "fpr"
        },
        {
          text: this.$t("PlayerStats.KAST"),
          value: "kast"
        },
        {
          text: this.$t("PlayerStats.ContribScore"),
          value: "contribution_score"
        },
        {
          text: this.$t("PlayerStats.MVP"),
          value: "mvp"
        }
      ];
    }
  },
  methods: {
    async useStreamOrStaticData() {
      // Template will contain v-rows/etc like on main Team page.
      let matchData = await this.GetMatchData(this.match_id);
      if (matchData.end_time == null) {
           //this.GetMapStatsStream(matchData);
           //this.GetMapPlayerStatsStream(matchData);
           this.getMapString(matchData);
           this.GetMapPlayerStats(matchData);
      }
      else {
           this.getMapString(matchData);
           this.GetMapPlayerStats(matchData);
      }
    },
    async retrieveStatsHelper(serverResponse, matchData) {
      if (typeof serverResponse == "string") return;
      let allMapIds = [];
      let totalMatchTeam = [];
      serverResponse.filter(item => {
        let i = allMapIds.findIndex(x => x == item.map_id);
        if (i <= -1) allMapIds.push(item.map_id);
        return null;
      });
      allMapIds.forEach(map_id => {
        totalMatchTeam.push(
          serverResponse.filter(stats => {
            return stats.map_id == map_id;
          })
        );
      });
      this.playerstats = totalMatchTeam;
      await this.playerstats.forEach((matchStats, idx) => {
        matchStats.forEach(async (player, pIdx) => {
          if (player.roundsplayed > 0) {
            let getRating = this.GetRating(
              player.kills,
              player.roundsplayed,
              player.deaths,
              player.k1,
              player.k2,
              player.k3,
              player.k4,
              player.k5
            );
            let adr = this.GetADR(player);
            let hsp = this.GetHSP(player);
            let kdr = this.GetKDR(player);
            let fpr = this.GetFPR(player);
            let teamNum;
            let newName;
            if (player.team_id) {
              teamNum = player.team_id == matchData.team1_id ? 1 : 2;
              newName =
                player.team_id == matchData.team1_id
                  ? matchData.team1_string
                  : matchData.team2_string;
            } else {
              // If we don't have a team ID, we must be pugging. Go based on
              // Team strings alone.
              teamNum = player.team_name == matchData.team1_string ? 1 : 2;
              newName = 
                player.team_name == matchData.team1_string
                  ? matchData.team1_string
                  : matchData.team2_string;
            }
            this.$set(
              this.playerstats[idx][pIdx],
              "Team",
              teamNum + " " + newName
            );
            this.$set(this.playerstats[idx][pIdx], "rating", getRating);
            this.$set(this.playerstats[idx][pIdx], "adr", adr);
            this.$set(this.playerstats[idx][pIdx], "hsp", hsp);
            this.$set(this.playerstats[idx][pIdx], "kdr", kdr);
            this.$set(this.playerstats[idx][pIdx], "fpr", fpr);
          }
        });
      });
      if (matchData.end_time != null) this.isFinished = true;
    },
    async GetMapPlayerStatsStream(matchData) {
      try {
        let sseClient = await this.GetEventPlayerStats(this.match_id);
        await sseClient.connect();
        await sseClient.on("playerstats", async message => {
          await this.retrieveStatsHelper(message, matchData);
        });
      } catch (error) {
        console.log("Our error: " + error);
      } finally {
        this.isLoading = false;
      }
      return;
    },
    async GetMapPlayerStats(matchData) {
      try {
        let res = await this.GetPlayerStats(this.match_id);
        await this.retrieveStatsHelper(res, matchData);
      } catch (error) {
        console.log("Our error: " + error);
      } finally {
        this.isLoading = false;
      }
      return;
    },
    async GetMapStatsStream(matchData) {
      try {
        let sseClient = await this.GetEventMapStats(this.match_id);
        await sseClient.connect();
        await sseClient.on("mapstats", async message => {
          await this.retrieveMapStatsHelper(message,matchData);
        });
      } catch (error) {
        console.log("Our error: " + error);
      } finally {
        this.isLoading = false;
      }
      return;
    },
    async getMapString(matchData) {
      try {
        let res = await this.GetMapStats(this.match_id);
        await this.retrieveMapStatsHelper(res, matchData);
      } catch (error) {
        console.log("Our error: " + error);
      } finally {
        this.isLoading = false;
      }
      return;
    },
    async retrieveMapStatsHelper(serverResponse, matchData) {
      if (typeof serverResponse == "string") return;
      await serverResponse.forEach((singleMapStat, index) => {
        if (!this.mapStats[index]) {
          this.$set(this.mapStats, index, {});
        }

        this.$set(this.mapStats[index], 'score', "Score: " +
          singleMapStat.team1_score +
          " " +
          this.GetScoreSymbol(
            singleMapStat.team1_score,
            singleMapStat.team2_score
          ) +
          " " +
          singleMapStat.team2_score);
        this.$set(this.mapStats[index], 'start', "Map Start: " + new Date(singleMapStat.start_time).toLocaleString());
        this.$set(this.mapStats[index], 'end', singleMapStat.end_time == null ?
          null :
          "Map End: " + new Date(singleMapStat.end_time).toLocaleString());
        this.$set(this.mapStats[index], 'map', "Map: " + singleMapStat.map_name);
        this.$set(this.mapStats[index], 'demo', singleMapStat.demoFile);
      });
      if (matchData.end_time != null) this.isFinished = true;
    }
  }
};
</script>