<template>
  <v-container class="team" fluid>
    <v-row class="pb-5">
      <v-col cols="12" class="flex-grow-1">
        <v-card>
          <TeamTable v-if="user" :user="user" :newTeam="newTeam" />
        </v-card>
      </v-col>
    </v-row>
    <v-row class="justify-center">
      <v-col lg="6" sm="12">
        <v-card>
          <v-card-title>{{ matchtitle }}</v-card-title>
          <MatchTable class="justify-center" />
        </v-card>
      </v-col>
    </v-row>
    <v-row class="justify-center">
      <v-col lg="6" sm="12">
        <v-card>
          <v-card-title>{{ playertitle }}</v-card-title>
          <PlayerLeaderboardTable :teamId="parseInt(this.$route.params.id)" />
        </v-card>
      </v-col>
    </v-row>
    
  </v-container>
</template>

<script>
// @ is an alias to /src
import TeamTable from "@/components/TeamTable";
import MatchTable from "@/components/MatchesTableNoLimits";
import PlayerLeaderboardTable from "@/components/PlayerLeaderboardTable.vue";
export default {
  name: "Teams",
  components: {
    TeamTable,
    MatchTable,
    PlayerLeaderboardTable
  },
  data() {
    return {
      newTeam: false,
      matchtitle: "Recent Matches",
      playertitle: "Players Statistique",
      user: {
        admin: false,
        steam_id: "",
        id: -1,
        super_admin: false,
        name: "",
        small_image: "",
        medium_image: "",
        large_image: ""
      }
    };
  },
  async created() {
    this.user = await this.IsLoggedIn();
    if (this.$route.params.id == "create") this.newTeam = true;
  }
};
</script>
