<template>
  <v-container fill-height>
    <div class="d-flex flex-column" style="height:100%; width:100%">
      <v-card>
        <v-text-field
          v-model="searchText"
          label="Search by username"
          :append-outer-icon="mdiMagnify"
          outlined
          style="width: 50%; left: 25%"
          centered
          @click:append-outer="reloadList();"
        />

        <v-card-actions class="justify-center">
          <v-btn @click="tabClick(0)">
            Applications
            <v-icon>{{ mdiPencilPlus }}</v-icon>
          </v-btn>

          <v-btn @click="tabClick(1)">
            Accepted
            <v-icon>{{ mdiCheckBold }}</v-icon>
          </v-btn>

          <v-btn @click="tabClick(2)">
            Rejected
            <v-icon>{{ mdiCloseThick }}</v-icon>
          </v-btn>

          <v-btn @click="tabClick(3)">
            Banned
            <v-icon>{{ mdiGavel }}</v-icon>
          </v-btn>
        </v-card-actions>
      </v-card>

      <v-card style="padding-top: 10px">
        <v-simple-table>
          <template #default>
            <thead>
              <tr>
                <th
                  v-for="header in headerList[tabIndex]"
                  :key="header"
                  class="text-left"
                >
                  {{ header }}
                </th>
              </tr>
            </thead>
            <tbody>
              <template
                v-for="(entry, index) in entryList"
              >
                <tr :key="entry.id">
                  <td
                    v-for="header in headerList[tabIndex]"
                    :key="header"
                    style="cursor: pointer"
                    @click="entryClick(index);"
                  >
                    {{ entry[header] }}
                  </td>
                </tr>
                <tr v-if="index === selectedIndex" :key="index">
                  <td :colspan="headerList[tabIndex].length">
                    <v-container v-if="tabIndex === 0">
                      <v-row>
                        <v-col cols="10">
                          <v-textarea
                            label="Note"
                            :value="entry.note"
                            readonly
                          />
                        </v-col>
                        <v-col cols="2">
                          <v-card class="d-flex flex-column" height="100%">
                            <v-btn>
                              Accept
                            </v-btn>
                            <v-btn style="margin-top:auto; margin-bottom:10px">
                              Reject
                            </v-btn>
                            <v-btn @click="selectedIndex=-1;">
                              Close
                            </v-btn>
                          </v-card>
                        </v-col>
                      </v-row>
                    </v-container>

                    <v-card-actions v-if="tabIndex === 1" class="justify-space-around">
                      <v-btn>
                        Ban
                      </v-btn>
                      <v-btn>
                        Revoke
                      </v-btn>
                      <v-btn @click="selectedIndex=-1;">
                        Close
                      </v-btn>
                    </v-card-actions>

                    <v-card-actions v-if="tabIndex === 2" class="justify-space-around">
                      <v-btn>
                        Accept
                      </v-btn>
                      <v-btn>
                        Change reviewer note
                      </v-btn>
                      <v-btn @click="selectedIndex=-1;">
                        Close
                      </v-btn>
                    </v-card-actions>

                    <v-container v-if="tabIndex === 3">
                      <v-textarea
                        label="Ban reason"
                        :value="entry.note"
                        readonly
                      />
                      <v-card-actions class="justify-space-around">
                        <v-btn>
                          Unban
                        </v-btn>
                        <v-btn>
                          Revoke TL privilege
                        </v-btn>
                        <v-btn>
                          Change ban reason
                        </v-btn>
                        <v-btn>
                          Change ban time
                        </v-btn>
                        <v-btn @click="selectedIndex=-1;">
                          Close
                        </v-btn>
                      </v-card-actions>
                    </v-container>
                  </td>
                </tr>
              </template>
            </tbody>
          </template>
        </v-simple-table>
      </v-card>
    </div>

    <!---------   NEXUS MODAL ---------

    -->
    <v-dialog
      v-model="modalNexus"
      max-width="600px"
    >
      <!---------    ADD PROFILE     --------->
      <v-card v-if="modalMode === 1">
        <v-container />
      </v-card>
    </v-dialog>
    <!--========   NEXUS MODAL =======-->
  </v-container>
</template>

<script lang="ts">
// import backendApi from "@/utils/backend-api";
import { mdiPencilPlus, mdiCheckBold, mdiCloseThick, mdiGavel, mdiMagnify } from "@mdi/js";

export default {
    name: "Tlmanager",
    metaInfo() {
        return {
            get title() {
                return "TLManager - Holodex";
            },
        };
    },
    components: {
    },
    data() {
        return {
            mdiPencilPlus,
            mdiCheckBold,
            mdiCloseThick,
            mdiGavel,
            mdiMagnify,
            tabIndex: 0,
            selectedIndex: -1,
            headerList: [
                ["id", "username"],
                ["id", "username"],
                ["id", "username", "note"],
                ["id", "username", "bantime"],
            ],
            entryList: [],
            searchText: "",
            modalNexus: false,
            modalMode: 0,
        };
    },
    computed: {
    },
    mounted() {
        this.reloadList();
    },
    methods: {
        tabClick(idx) {
            if (idx !== this.tabIndex) {
                this.searchText = "";
                this.tabIndex = idx;
                this.reloadList();
            }
        },
        reloadList() {
            this.selectedIndex = -1;
            this.entryList = [];
            for (let i = 0; i < 10; i = +1) {
                this.entryList.push({
                    id: i,
                    username: `username ${i}`,
                    note: `note ${i}`,
                    bantime: new Date(Date.now() + i * 1000 * 50).toLocaleString(),
                });
            }
            /*
            var stat = "";
            switch (this.tabIndex) {
                case 0:
                    stat = "APPLIED";
                    break;
                case 1:
                    stat = "OK";
                    break;
                case 2:
                    stat = "REJECTED";
                    break;
                case 3:
                    stat = "BANNED";
                    break;
            }

            backendApi.getTLApplications({
                username: this.searchText,
                status: stat
            }).then(({ data, status }) => {
                if ((status === 200) && (Array.isArray(data))){
                    this.entryList = data.map((e) => {
                        if (e.status !== "BANNED") {
                            e.bantime = new Date(e.bantime).toLocaleString();
                        }
                        return (e);
                    });
                } else {
                    console.log("Err response not array");
                }
            }).catch((err) => {
                console.log(err);
            });
            */
        },
        entryClick(idx) {
            this.selectedIndex = idx;
        },
    },
};
</script>
