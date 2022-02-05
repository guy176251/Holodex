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
                    {{ (header !== 'bantime') ? entry[header] : new Date(entry[header]).toLocaleString() }}
                  </td>
                </tr>
                <tr v-if="index === selectedIndex" :key="'control' + index">
                  <td :colspan="headerList[tabIndex].length">
                    <v-container v-if="tabIndex === 0">
                      <v-row>
                        <v-col cols="10">
                          <v-textarea
                            label="Note"
                            :value="entry.applynote"
                            readonly
                          />
                        </v-col>
                        <v-col cols="2">
                          <v-card class="d-flex flex-column" height="100%">
                            <v-btn :color="(snackBar) ? 'warning' : 'success'" @click="acceptApplication();">
                              Accept
                            </v-btn>
                            <v-btn style="margin-top:auto; margin-bottom:10px" @click="openRejectModal();">
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
                      <v-btn @click="openBanModal();">
                        Ban
                      </v-btn>
                      <v-btn @click="openRejectModal();">
                        Revoke
                      </v-btn>
                      <v-btn @click="selectedIndex=-1;">
                        Close
                      </v-btn>
                    </v-card-actions>

                    <v-container v-if="tabIndex === 2">
                      <v-row>
                        <v-col cols="8">
                          <v-textarea
                            label="Note"
                            :value="entry.applynote"
                            readonly
                          />
                        </v-col>
                        <v-col cols="4">
                          <v-card class="d-flex flex-column" height="100%">
                            <v-btn :color="(snackBar) ? 'warning' : 'success'" @click="acceptApplication();">
                              Accept
                            </v-btn>
                            <v-btn style="margin-top:auto; margin-bottom:10px" @click="openChangeRejectNoteModal();">
                              Change reviewer note
                            </v-btn>
                            <v-btn @click="selectedIndex=-1;">
                              Close
                            </v-btn>
                          </v-card>
                        </v-col>
                      </v-row>
                    </v-container>

                    <v-container v-if="tabIndex === 3">
                      <v-textarea
                        label="Ban reason"
                        :value="entry.note"
                        readonly
                      />
                      <v-card-actions class="justify-space-around">
                        <v-btn :color="(snackBar) ? 'warning' : 'success'" @click="sendUnban();">
                          Unban
                        </v-btn>
                        <v-btn @click="openRejectModal();">
                          Revoke TL privilege
                        </v-btn>
                        <v-btn @click="openChangeBanModal();">
                          Change ban reason/time
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

    <v-snackbar
      v-model="snackBar"
      timeout="5000"
    >
      <v-card-title>
        Click one more time to confirm.
      </v-card-title>
    </v-snackbar>

    <!---------   NEXUS MODAL ---------
    1. Privilege check
    2. Reject application
    3. Ban
    4. Change rejection note
    -->
    <v-dialog
      v-model="modalNexus"
      max-width="600px"
      :persistent="!loggedIn"
    >
      <!---------     PRIVILEGE CHECKING     --------->
      <v-card v-if="modalMode === 1">
        <v-container>
          <v-card-title>
            {{ loginStatusText }}
          </v-card-title>

          <v-card-actions>
            <v-btn href="/">
              Back
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------     REJECT APPLICATION     --------->
      <v-card v-if="modalMode === 2">
        <v-container>
          <v-card-title>
            Reject\revoke application.
          </v-card-title>

          <v-text-field
            v-model="modalTextInput"
            label="Reason"
            outlined
            rounded
          />

          <v-card-actions>
            <v-btn @click="modalNexus=false;">
              Back
            </v-btn>

            <v-btn style="margin-left:auto" @click="rejectApplication();">
              Reject/revoke
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------     BAN     --------->
      <v-card v-if="modalMode === 3">
        <v-container>
          <v-card-title>
            Ban translator (default is 3 days)
          </v-card-title>

          <v-row>
            <v-date-picker
              v-model="modalDate"
              style="margin-right: 10px;"
            />

            <v-time-picker
              v-model="modalTime"
              format="24hr"
              scrollable
            />
          </v-row>

          <v-text-field
            v-model="modalTextInput"
            style="margin-top: 15px"
            label="Ban reason"
            outlined
            rounded
          />

          <v-card-actions>
            <v-btn @click="modalNexus=false;">
              Back
            </v-btn>

            <v-btn style="margin-left:auto" @click="sendBan();">
              Ban
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------     REJECT APPLICATION     --------->
      <v-card v-if="modalMode === 4">
        <v-container>
          <v-card-title>
            Change reviewer note.
          </v-card-title>

          <v-text-field
            v-model="modalTextInput"
            label="Reason"
            outlined
            rounded
          />

          <v-card-actions>
            <v-btn @click="modalNexus=false;">
              Back
            </v-btn>

            <v-btn style="margin-left:auto" @click="rejectApplication();">
              Save
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------     BAN     --------->
      <v-card v-if="modalMode === 5">
        <v-container>
          <v-card-title>
            Change ban
          </v-card-title>

          <v-row>
            <v-date-picker
              v-model="modalDate"
              style="margin-right: 10px;"
            />

            <v-time-picker
              v-model="modalTime"
              format="24hr"
              scrollable
            />
          </v-row>

          <v-text-field
            v-model="modalTextInput"
            style="margin-top: 15px"
            label="Ban reason"
            outlined
            rounded
          />

          <v-card-actions>
            <v-btn @click="modalNexus=false;">
              Back
            </v-btn>

            <v-btn style="margin-left:auto" @click="sendBan();">
              Save
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>
    </v-dialog>
    <!--========   NEXUS MODAL =======-->
  </v-container>
</template>

<script lang="ts">
import backendApi from "@/utils/backend-api";
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

            modalNexus: true,
            modalMode: 1,
            modalTextInput: "",
            modalTime: null,
            modalDate: null,

            snackBar: false,

            // ---- PRIVILIGE CHECK ----
            loginStatusText: "Checking login status...",
            loggedIn: false,
        };
    },
    computed: {
        userdata() {
            return this.$store.state.userdata;
        },
        user() {
            return this.$store.state.userdata.user;
        },
    },
    mounted() {
        this.checkPrivilege();
    },
    methods: {
        async checkPrivilege() {
            const check = await backendApi.loginIsValid(this.userdata.jwt);
            if (check === false) {
                this.$store.dispatch("logout");
                this.loginStatusText = "Not logged in";
            } else if (check.data && check.data.id) {
                this.$store.commit("setUser", { user: check.data, jwt: this.userdata.jwt });
                if (check.data.role === "admin") {
                    this.loggedIn = true;
                    this.modalNexus = false;
                    this.reloadList();
                } else {
                    this.loginStatusText = "Only admin or mod can access this page";
                }
            }
        },
        tabClick(idx) {
            if (idx !== this.tabIndex) {
                this.searchText = "";
                this.tabIndex = idx;
                this.snackBar = false;
                this.reloadList();
            }
        },
        reloadList() {
            this.selectedIndex = -1;
            /*
            this.entryList = [];
            for (let i = 0; i < 10; i = i + 1) {
              this.entryList.push({
                id: i,
                username: `username ${i}`,
                note: `note ${i}`,
                applynote: `applynote ${i}`,
                bantime: Date.now() + i * 1000 * 50,
              });
            }
            */

            let stat = "";
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
                default:
                    stat = "";
                    break;
            }

            backendApi.getTLApplications({
                username: this.searchText,
                status: stat,
            }).then(({ data, status }) => {
                if ((status === 200) && (Array.isArray(data))) {
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
        },
        entryClick(idx) {
            this.selectedIndex = idx;
            this.snackBar = false;
        },
        acceptApplication() {
            if (this.snackBar) {
                backendApi.changeTLStatus({
                    id: this.entryList[this.selectedIndex].id,
                    status: "OK",
                    note: "",
                }).then(({ status, data }) => {
                    if (status === 200) {
                        this.entryList.splice(this.selectedIndex, 1);
                        this.selectedIndex = -1;
                        this.snackBar = false;
                    } else {
                        console.log(`ERROR ${data}`);
                    }
                }).catch((err) => {
                    console.log(`ERROR ${err}`);
                });
            } else {
                this.snackBar = true;
            }
        },
        openRejectModal() {
            this.modalNexus = true;
            this.modalMode = 2;
            this.modalTextInput = "";
        },
        rejectApplication() {
            backendApi.changeTLStatus({
                id: this.entryList[this.selectedIndex].id,
                status: "REJECTED",
                note: this.modalTextInput,
            }).then(({ status, data }) => {
                if (status === 200) {
                    if (this.modalMode === 2) {
                        this.entryList.splice(this.selectedIndex, 1);
                        this.selectedIndex = -1;
                    } else {
                        this.entryList[this.selectedIndex].note = this.modalTextInput;
                    }
                    this.modalNexus = false;
                } else {
                    console.log(`ERROR ${data}`);
                }
            }).catch((err) => {
                console.log(`ERROR ${err}`);
            });
        },
        openBanModal() {
            this.modalNexus = true;
            this.modalMode = 3;
            this.modalTextInput = "";
            const timeString = new Date(Date.now() + 3 * 24 * 60 * 60 * 1000);
            this.modalDate = `${timeString.getFullYear().toString()}-${(timeString.getMonth() + 1).toString()}-${timeString.getDate().toString()}`;
            this.modalTime = `${timeString.getHours().toString()}:${timeString.getMinutes().toString()}`;
        },
        sendBan() {
            const bantime = new Date(
                this.modalDate.split("-")[0],
                this.modalDate.split("-")[1],
                this.modalDate.split("-")[2],
                this.modalTime.split(":")[0],
                this.modalTime.split(":")[1],
                0,
                0,
            );
            backendApi.changeTLStatus({
                id: this.entryList[this.selectedIndex].id,
                status: "BANNED",
                note: this.modalTextInput,
                bantime: bantime.getTime(),
            }).then(({ status, data }) => {
                if (status === 200) {
                    if (this.modalMode === 3) {
                        this.entryList.splice(this.selectedIndex, 1);
                        this.selectedIndex = -1;
                    } else {
                        this.entryList[this.selectedIndex].note = this.modalTextInput;
                        this.entryList[this.selectedIndex].bantime = bantime.getTime();
                    }
                    this.modalNexus = false;
                } else {
                    console.log(`ERROR ${data}`);
                }
            }).catch((err) => {
                console.log(`ERROR ${err}`);
            });
        },
        openChangeRejectNoteModal() {
            this.modalNexus = true;
            this.modalMode = 4;
            this.modalTextInput = this.entryList[this.selectedIndex].note;
        },
        sendUnban() {
            if (this.snackBar) {
                backendApi.changeTLStatus({
                    id: this.entryList[this.selectedIndex].id,
                    status: "OK",
                    note: "",
                }).then(({ status, data }) => {
                    if (status === 200) {
                        this.entryList.splice(this.selectedIndex, 1);
                        this.selectedIndex = -1;
                        this.snackBar = false;
                    } else {
                        console.log(`ERROR ${data}`);
                    }
                }).catch((err) => {
                    console.log(`ERROR ${err}`);
                });
            } else {
                this.snackBar = true;
            }
        },
        openChangeBanModal() {
            this.modalNexus = true;
            this.modalMode = 5;
            this.modalTextInput = this.entryList[this.selectedIndex].note;
            const timeString = new Date(this.entryList[this.selectedIndex].bantime); this.modalDate = `${timeString.getFullYear().toString()}-${(timeString.getMonth() + 1).toString()}-${timeString.getDate().toString()}`;
            this.modalTime = `${timeString.getHours().toString()}:${timeString.getMinutes().toString()}`;
        },
    },
};
</script>
