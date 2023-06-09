<!-- Kalipo B.V. - the DAO platform for business & societal impact
 * Copyright (C) 2022 Peter Nobels and Matthias van Dijk
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation; either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <https://www.gnu.org/licenses/>.
-->
<template>
  <v-container>
    <v-row class="mt-2 ml-1" style="justify-content: left; align-items: center">
      <div>
        <v-text-field
          v-if="poas.length != null"
          solo
          label="Search a poa"
          append-icon="mdi-magnify"
          class="mt-4 mr-4"
          style="max-width: 250px"
          v-model="search"
        ></v-text-field>
      </div>
      <div>
        <v-btn
          class="mb-2 pa-6"
          :disabled="authorizedNewPoa"
          text
          outlined
          @click="dialog = !dialog"
          >Create POA template</v-btn
        >
      </div>
    </v-row>
    <v-row class="">
      <v-col xs="12" sm="6" md="4" lg="3" v-for="(poa, i) in filtered" :key="i">
        <PoaCard :poa="poa"></PoaCard>
      </v-col>
    </v-row>

    <v-dialog v-model="dialog" max-width="500px">
      <PoaCreateDialog :autonId="autonId" :auton="auton"></PoaCreateDialog>
    </v-dialog>
  </v-container>
</template>
<script>
export default {
  layout: "auton",
  data: () => ({
    dialog: false,
    poas: [],
    auton: null,
    autonId: null,
    authorizedNewPoa: true,
    search: "",
  }),
  computed: {
    account() {
      return this.$store.state.wallet.account;
    },
    unlocked() {
      return this.$store.state.wallet.unlocked;
    },
    filtered() {
      return this.poas.filter((poa) =>
        poa.name.toLowerCase().includes(this.search.toLowerCase())
      );
    },
  },
  methods: {
    async authorized() {
      if (this.unlocked) {
        const memberships = this.$store.state.wallet.account.memberships;
        memberships.forEach(async (element) => {
          const mship = await this.$invoke("membership:getByID", {
            id: element,
          });
          if (
            mship.result.autonId == this.autonId &&
            mship.result.role == "FULL_MEMBER"
          ) {
            this.authorizedNewPoa = false;
          }
        });
      } else {
        return true;
      }
    },
  },
  async mounted() {
    // this.$nuxt.$emit("Auton-setPage", "poas");

    const autonIdParam = this.$route.params.autonId.replaceAll("_", " ");

    const autonIdWrapper = await this.$invoke("auton:getAutonIdByName", {
      name: autonIdParam,
    });
    if (autonIdWrapper.result === null) {
      this.auton = null;
      this.error = "Auton not found: " + autonIdParam;
    } else {
      this.autonId = autonIdWrapper.result.id;
      const autonWrapper = await this.$invoke("auton:getByID", {
        id: autonIdWrapper.result.id,
      });
      this.auton = autonWrapper.result;

      for (let index = 0; index < this.auton.poas.length; index++) {
        const poaId = this.auton.poas[index];

        const poaWrapper = await this.$invoke("poa:getByID", {
          id: poaId,
        });

        this.poas.push(poaWrapper.result);
      }
    }

    this.authorized();
  },
};
</script>
<style lang=""></style>
