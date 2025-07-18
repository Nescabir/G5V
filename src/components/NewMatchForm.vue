<template>
  <v-card class="mx-auto" fluid>
    <v-card-title class="title font-weight-regular justify-space-between">
      <span>{{ currentTitle }}</span>
      <v-avatar
        color="primary lighten-2"
        class="subheading white--text"
        size="24"
        v-text="step"
      ></v-avatar>
    </v-card-title>
    <v-window v-model="step">
      <v-form ref="newMatchForm">
        <v-window-item :value="1">
          <v-col cols="12">
            <v-select
              v-model="selectedServer"
              :items="servers"
              item-text="display_name"
              item-value="id"
              :rules="[v => v != -1 || $t('misc.Required')]"
              :label="$t('CreateMatch.ServerLabel')"
              required
              ref="newServer"
            >
              <template v-slot:item="{ item }">
                {{ item.display_name }} {{ item.flag }}
              </template>
            </v-select>
          </v-col>
          <v-card-text>
            {{ $t("CreateMatch.ServerNote") }}
          </v-card-text>
          <v-col cols="12">
            <v-btn color="primary" @click="newDialog = true">
              {{ $t("misc.Create") }} {{ $t("CreateMatch.ServerSelect") }}
            </v-btn>
          </v-col>
        </v-window-item>

        <v-window-item :value="2">
          <v-col cols="12">
            <v-select
              v-model="selectedSeason"
              :items="seasons"
              item-text="name"
              item-value="id"
              :label="$t('CreateMatch.SeasonLabel')"
              ref="newServer"
            />
          </v-col>
          <v-card-text>
            <strong>
              {{ $t("CreateMatch.SeasonNote") }}
            </strong>
          </v-card-text>
        </v-window-item>

        <v-window-item :value="3">
          <div class="pa-4 text-center">
            <v-combobox
              v-model="newMatchData.team1"
              :items="teams"
              item-text="name"
              item-value="id"
              :rules="[
                v => !!v || $t('CreateMatch.TeamRequired'),
                () =>
                  newMatchData.team1.id != newMatchData.team2.id ||
                  $t('CreateMatch.TeamCannotBeEqual')
              ]"
              :label="$t('CreateMatch.FormTeam1')"
              required
              solo
              clearable
              ref="teamOne"
            />
            <v-combobox
              v-model="newMatchData.team2"
              :items="teams"
              item-text="name"
              item-value="id"
              :rules="[
                v => !!v || $t('CreateMatch.TeamRequired'),
                () =>
                  newMatchData.team2.id != newMatchData.team1.id ||
                  $t('CreateMatch.TeamCannotBeEqual')
              ]"
              :label="$t('CreateMatch.FormTeam2')"
              required
              solo
              clearable
              ref="teamTwo"
            />
            <v-divider />
            <v-row class="justify-center">
              <v-col cols="2">
                <v-switch
                  v-model="newMatchData.wingman"
                  :label="$t('CreateMatch.Wingman')"
                  ref="wingman"
                />
              </v-col>
            </v-row>
            <v-divider />
            <v-row class="justify-center">
              <v-col cols="12">
                <strong>{{ $t("CreateMatch.FormSeriesType") }}</strong>
              </v-col>
              <v-radio-group v-model="newMatchData.maps_to_win" row>
                <v-col lg="3" sm="12">
                  <v-radio :label="$t('CreateMatch.BestOf') + 1" :value="1" />
                </v-col>
                <v-col lg="3" sm="12">
                  <v-radio :label="$t('CreateMatch.BestOf') + 2" :value="2" />
                </v-col>
                <v-col lg="3" sm="12">
                  <v-radio :label="$t('CreateMatch.BestOf') + 3" :value="3" />
                </v-col>
                <v-col lg="3" sm="12">
                  <v-radio :label="$t('CreateMatch.BestOf') + 5" :value="5" />
                </v-col>
                <v-col lg="3" sm="12" offset-lg="5">
                  <v-radio :label="$t('CreateMatch.BestOf') + 7" :value="7" />
                </v-col>
              </v-radio-group>
            </v-row>
            <v-divider />
            <div>
              <strong>
                {{ $t("CreateMatch.FormMapPool") }}
                <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn v-bind="attrs" v-on="on" x-small fab icon>
                      <v-icon>mdi-information</v-icon>
                    </v-btn>
                  </template>
                  <span>{{ $t("CreateMatch.FormMapExplanation") }}</span>
                </v-tooltip>
              </strong>
            </div>
            <v-row class="justify-center">
              <v-col lg="1" sm="12" v-for="maps in MapList" :key="maps.id">
                <v-checkbox
                  v-model="newMatchData.map_pool"
                  :value="maps.map_name"
                  :label="maps.map_display_name"
                  :rules="[
                    () =>
                      newMatchData.map_pool.length > 0 ||
                      $t('CreateMatch.MapChoiceError'),
                    () =>
                      newMatchData.map_pool.length >
                        newMatchData.maps_to_win - 1 ||
                      $t('CreateMatch.MapNotEnough')
                  ]"
                />
              </v-col>
            </v-row>
            <v-divider />
            <v-row class="justify-center">
              <v-col lg="3" md="12" sm="12">
                {{ $t("CreateMatch.PlayersPerTeam") }}
                <v-slider
                  v-model="newMatchData.players_per_team"
                  single-line
                  :min="1"
                  :max="5"
                  :thumb-size="24"
                  thumb-label
                  ticks="always"
                  tick-size="4"
                  step="1"
                />
              </v-col>
              <v-col lg="3" md="12" sm="12">
                {{ $t("CreateMatch.MinPlayersReady") }}
                <v-slider
                  v-model="newMatchData.min_players_to_ready"
                  single-line
                  :min="1"
                  :max="5"
                  :thumb-size="24"
                  thumb-label
                  ticks="always"
                  tick-size="4"
                  step="1"
                />
              </v-col>
              <v-col lg="3" md="12" sm="12">
                {{ $t("CreateMatch.SpectatorsToReady") }}
                <v-slider
                  v-model="newMatchData.min_spectators_to_ready"
                  single-line
                  :min="0"
                  :thumb-size="24"
                  thumb-label
                  ticks="always"
                  tick-size="4"
                  step="1"
                />
              </v-col>
            </v-row>
            <v-divider />
            <v-col cols="12" class="text-center text-h6">
              {{ $t("CreateMatch.Spectators") }}
            </v-col>
            <v-row class="justify-center">
              <v-col cols="12">
                <v-combobox
                  v-model="newMatchData.spectators"
                  :label="$t('CreateMatch.Spectators')"
                  ref="matchspecs"
                  :hint="$t('Team.AuthHint')"
                  multiple
                  chips
                  deletable-chips
                />
              </v-col>
            </v-row>
            <v-divider />
            <v-row class="justify-center">
              <v-col cols="2">
                <v-switch
                  v-model="newMatchData.skip_veto"
                  :label="$t('CreateMatch.SkipVeto')"
                  ref="skipveto"
                />
              </v-col>
            </v-row>
            <v-row class="justify-center" v-if="newMatchData.skip_veto">
              <v-col
                lg="3"
                md="12"
                sm="12"
                v-for="(entity, index) in newMatchData.maps_to_win"
                :key="index"
              >
                <v-col class="text-left text-h6">
                  {{
                    $t("CreateMatch.MapSides", {
                      map:
                        newMatchData.map_pool[index] == null
                          ? entity
                          : newMatchData.map_pool[index]
                    })
                  }}
                </v-col>
                <v-radio-group row v-model="newMatchData.map_sides[index]">
                  <v-col lg="12" sm="12" align-self="center">
                    <v-radio
                      :label="$t('CreateMatch.MapSidesTeam1CT')"
                      :value="'team1_ct'"
                    />
                  </v-col>
                  <v-col lg="12" sm="12" align-self="center">
                    <v-radio
                      :label="$t('CreateMatch.MapSidesTeam2CT')"
                      :value="'team1_t'"
                    />
                  </v-col>
                  <v-col lg="12" sm="12" align-self="center">
                    <v-radio
                      :label="$t('CreateMatch.MapSidesKnife')"
                      :value="'knife'"
                    />
                  </v-col>
                </v-radio-group>
              </v-col>
            </v-row>
            <v-row class="justify-center">
              <v-radio-group
                v-model="newMatchData.side_type"
                row
                class="justify-center"
              >
                <v-col lg="4" sm="13" align-self="center">
                  <v-radio
                    :label="$t('CreateMatch.KnifeDefault')"
                    :value="'standard'"
                  />
                </v-col>
                <v-col lg="4" sm="12" align-self="center">
                  <v-radio
                    :label="$t('CreateMatch.KnifeNever')"
                    :value="'never_knife'"
                  />
                </v-col>
                <v-col lg="4" sm="12" align-self="center">
                  <v-radio
                    :label="$t('CreateMatch.KnifeAlways')"
                    :value="'always_knife'"
                  />
                </v-col>
              </v-radio-group>
            </v-row>
            <v-divider />
            <v-col cols="12" class="text-center text-h6">
              <strong>{{ $t("CreateMatch.ConvarTitle") }}</strong>
            </v-col>
            <v-row class="justify-center">
              <v-col cols="12">
                <v-combobox
                  v-model="newMatchData.cvars"
                  :label="$t('CreateMatch.FormCVARS')"
                  ref="CVARs"
                  multiple
                  chips
                  deletable-chips
                />
              </v-col>
            </v-row>
          </div>
        </v-window-item>
      </v-form>
    </v-window>
    <v-divider></v-divider>
    <v-card-actions>
      <v-btn :disabled="step === 1" text @click="checkValidation(false)">
        {{ $t("misc.Back") }}
      </v-btn>
      <v-spacer />
      <v-btn
        color="primary"
        depressed
        @click="callCreateMatch"
        v-if="step === 3"
        :loading="isLoading"
      >
        {{ $t("misc.Create") }}
      </v-btn>
      <v-btn
        v-else
        :disabled="step === 3"
        color="primary"
        depressed
        @click="checkValidation"
      >
        {{ $t("misc.Next") }}
      </v-btn>
    </v-card-actions>
    <ServerDialog
      v-model="newDialog"
      :serverInfo="newServer"
      :title="$t('ServerCreate.NewServerTitle')"
      @is-new-server="ReloadServers"
    />
    <v-bottom-sheet v-model="responseSheet" inset persistent>
      <v-sheet class="text-center" height="200px">
        <v-btn class="mt-6" text color="success" @click="GoToMatch">
          {{ $t("misc.Close") }}
        </v-btn>
        <div class="my-3">
          {{ response }}
        </div>
      </v-sheet>
    </v-bottom-sheet>
  </v-card>
</template>

<script>
import ServerDialog from "./ServerDialog";
export default {
  props: {
    user: Object
  },
  name: "CreateMatch",
  components: {
    ServerDialog
  },
  data: () => ({
    step: 1,
    servers: [],
    teams: [],
    seasons: [],
    selectedServer: -1,
    selectedSeasonObject: {},
    newServer: {},
    selectedSeason: -1,
    newMatchData: {
      min_players_to_ready: 5,
      min_spectators_to_ready: 0,
      players_per_team: 5,
      maps_to_win: 1,
      skip_veto: false,
      map_pool: [],
      cvars: [],
      veto_first: "team1",
      spectators: [],
      side_type: "standard",
      map_sides: [],
      wingman: false
    },
    selectedTeams: [],
    newDialog: false,
    response: "",
    responseSheet: false,
    newMatchId: null,
    isLoading: false,
    MapList: []
  }),
  computed: {
    currentTitle() {
      switch (this.step) {
        case 1:
          return this.$t("CreateMatch.FormServer");
        case 2:
          return this.$t("CreateMatch.FormSeason");
        case 3:
          return this.$t("CreateMatch.FormDetail");
        default:
          return this.$t("CreateMatch.FormError");
      }
    }
  },
  watch: {
    selectedSeason(val) {
      let arrIndex = this.seasons
        .map(obj => {
          return obj.id;
        })
        .indexOf(val);
      this.selectedSeasonObject = this.seasons[arrIndex];
    },
    step(val) {
      if (val == 3) {
        if (this.selectedSeasonObject.cvars != null) {
          let seasonCvars = this.selectedSeasonObject.cvars;
          this.newMatchData.min_players_to_ready =
            seasonCvars.min_players_to_ready == null
              ? 5
              : parseInt(seasonCvars.min_players_to_ready);
          this.newMatchData.min_spectators_to_ready =
            seasonCvars.min_spectators_to_ready == null
              ? 0
              : parseInt(seasonCvars.min_spectators_to_ready);
          this.newMatchData.side_type =
            seasonCvars.side_type == null ? "standard" : seasonCvars.side_type;
          this.newMatchData.players_per_team =
            seasonCvars.players_per_team == null
              ? 5
              : parseInt(seasonCvars.players_per_team);
          this.newMatchData.maps_to_win =
            seasonCvars.maps_to_win == null
              ? 1
              : parseInt(seasonCvars.maps_to_win);
          this.newMatchData.skip_veto =
            seasonCvars.skip_veto == null || seasonCvars.skip_veto == 0
              ? false
              : true;
          this.newMatchData.map_pool =
            seasonCvars.map_pool.length < 1
              ? []
              : seasonCvars.map_pool.trim().split(" ");
          this.newMatchData.spectators =
            seasonCvars.spectators.length < 1
              ? null
              : seasonCvars.spectators.trim().split(" ");
          this.newMatchData.map_sides =
            seasonCvars.map_sides.length < 1
              ? []
              : seasonCvars.map_sides.trim().split(" ");
          this.newMatchData.wingman =
            seasonCvars.wingman == null || seasonCvars.wingman == 0
              ? false
              : true;
          //Delete all used get prepare custom CVARs.
          delete seasonCvars.min_players_to_ready;
          delete seasonCvars.min_spectators_to_ready;
          delete seasonCvars.players_per_team;
          delete seasonCvars.maps_to_win;
          delete seasonCvars.wingman;
          delete seasonCvars.skip_veto;
          delete seasonCvars.map_pool;
          delete seasonCvars.side_type;
          delete seasonCvars.spectators;
          delete seasonCvars.map_sides;
          // Now set Match CVARs. These will be converted back on submit.
          let tmpCvarArr = [];
          for (var obj in seasonCvars)
            tmpCvarArr.push(obj + " " + seasonCvars[obj]);
          this.newMatchData.cvars = tmpCvarArr;
        }
      }
    }
  },
  async created() {
    this.servers = await this.GetAllAvailableServers();
    if (typeof this.servers != "string") {
      this.servers.sort((a, b) => {
        return a.user_id - this.user.id || b.public_server - a.public_server;
      });
    }
    //if (this.IsAnyAdmin(this.user)) this.teams = await this.GetAllTeams();
    if (this.IsAnyAdmin(this.user)) this.teams = await this.GetPublicTeams();
    else {
      let tmpPublicTeams = await this.GetAllTeams();
      this.teams = await this.GetMyTeams();
      tmpPublicTeams.forEach(async team => {
        if (typeof this.teams === "string") this.teams = [];
        if (team.public_team == 1) this.teams.push(team);
      });
    }
    this.teams.sort((a, b) => {
      return a.user_id - this.user.id || a.public_team - b.public_team;
    });
    this.seasons = await this.GetMyAvailableSeasons();
    if (typeof this.seasons == "string") this.seasons = [];
    this.MapList = await this.GetUserEnabledMapList(this.user.id);
  },
  methods: {
    async ReloadServers() {
      this.servers = await this.GetAllAvailableServers();
      let arrIndex = this.servers
        .map(obj => {
          return obj.ip_string + " " + obj.port + " " + obj.user_id;
        })
        .indexOf(
          this.newServer.ip_string +
            " " +
            this.newServer.port +
            " " +
            this.user.id
        );
      this.selectedServer = this.servers[arrIndex].id;
      this.newServer = {};
    },
    checkValidation(moveForward = true) {
      if (this.$refs.newMatchForm.validate()) {
        if (moveForward) this.step++;
        else this.step--;
      }
    },
    async callCreateMatch() {
      if (this.$refs.newMatchForm.validate()) {
        this.isLoading = true;
        const splitStr = x => {
          const y = x.split(" ");
          let retVal;
          let key;
          let val;
          try {
            key = y[0].trim();
            y.splice(0, 1);
            val = y.join(" ").trim();
            if (!isNaN(val)) val = parseInt(val);
            retVal = { [key]: val };
          } catch (error) {
            retVal = { [key]: "" };
          }
          return retVal;
        };
        let newCvar = Object.assign(
          {},
          ...this.newMatchData.cvars.map(splitStr)
        );
        let matchInsertObj = [
          {
            server_id: this.selectedServer,
            team1_id: this.newMatchData.team1.id,
            team2_id: this.newMatchData.team2.id,
            season_id: this.selectedSeason == -1 ? null : this.selectedSeason,
            start_time: new Date()
              .toISOString()
              .slice(0, 19)
              .replace("T", " "),
            max_maps: this.newMatchData.maps_to_win,
            side_type: this.newMatchData.side_type,
            veto_mappool: this.newMatchData.map_pool.join(" "),
            match_cvars: newCvar,
            veto_first: this.newMatchData.veto_first,
            skip_veto: this.newMatchData.skip_veto,
            wingman: this.newMatchData.wingman,
            spectator_auths: this.newMatchData.spectators,
            min_players_to_ready: parseInt(
              this.newMatchData.min_players_to_ready
            ),
            players_per_team: parseInt(this.newMatchData.players_per_team),
            min_spectators_to_ready: parseInt(
              this.newMatchData.min_spectators_to_ready
            ),
            map_sides: this.newMatchData.map_sides.join(",")
          }
        ];
        try {
          let serverRes = await this.InsertMatch(matchInsertObj);
          if (serverRes.id != null)
            this.response = this.$t("CreateMatch.MessageRegisterSuccess");
          else this.response = serverRes.message;
          this.newMatchId = serverRes.id;
        } catch (error) {
          this.response = error.message;
          this.newMatchId = null;
        }
        this.responseSheet = true;
        this.isLoading = false;
        return;
      }
    },
    GoToMatch() {
      this.responseSheet = !this.responseSheet;
      this.response = "";
      console.log(this.newMatchId);
      if (this.newMatchId != null)
        this.$router.push({ name: `Match`, params: { id: this.newMatchId } });
    }
  }
};
</script>
