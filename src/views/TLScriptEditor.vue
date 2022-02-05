<template>
  <v-container fill-height>
    <v-card
      v-if="menuBool"
      class="TopMenu"
    >
      <v-container class="d-flex align-baseline">
        <v-btn
          elevation="4"
          color="secondary"
          @click="modalMode = 3; modalNexus = true; activeURLInput = activeURLStream;"
        >
          Load Video
        </v-btn>
        <v-btn
          elevation="4"
          color="secondary"
          style="margin-left:5px"
          @click="vidPlayer = false;"
        >
          Unload Video
        </v-btn>
      </v-container>
    </v-card>
    <div class="d-flex flex-column" style="height:100%; width:100%">
      <v-btn
        block
        elevation="4"
        color="primary"
        small
        @click="menuBool = !menuBool"
      >
        Menu
      </v-btn>
      <div class="d-flex align-stretch flex-row" style="height:100%" @click="menuBool = false">
        <v-card class="ChatOuterContainer grow" height="100%;" width="auto">
          <v-container class="ChatInnerContainer d-flex flex-column">
            <EnhancedEntry
              v-for="(dt, index) in entries"
              :key="index"
              :time="dt.Time"
              :stext="dt.SText"
              :cc="dt.CC"
              :oc="dt.OC"
            />
          </v-container>
          <v-card v-if="profileDisplay" class="ProfileListCard d-flex flex-column">
            <span v-for="(prf, index) in profile" :key="index"><span v-if="index === profileIdx">> </span>{{ (index + 1) + '. ' + prf.Name }}</span>
          </v-card>
        </v-card>
        <v-card
          v-if="vidPlayer"
          height="100%"
          width="50%"
          outlined
        >
          <v-card
            id="player"
            height="100%"
            width="100%"
          />
        </v-card>
      </div>

      <v-container
        @click="menuBool = false"
        @keydown.up.exact="profileUp()"
        @keydown.down.exact="profileDown()"
        @keydown.tab.exact.prevent="profileDown()"
        @keydown.shift.tab.exact.prevent="profileJumpToDefault()"
        @keydown.ctrl.49.exact.prevent="profileJump(0)"
        @keydown.ctrl.50.exact.prevent="profileJump(1)"
        @keydown.ctrl.51.exact.prevent="profileJump(2)"
        @keydown.ctrl.52.exact.prevent="profileJump(3)"
        @keydown.ctrl.53.exact.prevent="profileJump(4)"
        @keydown.ctrl.54.exact.prevent="profileJump(5)"
        @keydown.ctrl.55.exact.prevent="profileJump(6)"
        @keydown.ctrl.56.exact.prevent="profileJump(7)"
        @keydown.ctrl.57.exact.prevent="profileJump(8)"
      >
        <v-row class="align-baseline">
          <v-text-field
            v-model="inputString"
            @keypress.enter="addEntry()"
          />
          <v-btn style="margin-left:10px" @click="addEntry()">
            Enter
          </v-btn>
        </v-row>
        <v-row class="align-stretch">
          <v-col cols="2">
            <v-text-field
              v-model="profile[profileIdx].Prefix"
              label="Prefix"
              dense
              rounded
              outlined
            />
          </v-col>
          <v-col cols="2">
            <v-text-field
              v-model="profile[profileIdx].Suffix"
              label="Suffix"
              dense
              rounded
              outlined
            />
          </v-col>
          <v-checkbox
            v-model="profile[profileIdx].useCC"
            class="shrink"
            label="Font Colour : "
            hide-details
          />
          <v-btn
            class="ColourButton"
            small
            :style="{ background: profile[profileIdx].CC }"
            @click.stop="colourTemp = profile[profileIdx].CC; colourPick = 1; colourDialogue = true;"
          >
            {{ profile[profileIdx].CC }}
          </v-btn>
          <v-checkbox
            v-model="profile[profileIdx].useOC"
            label="Outline Colour : "
            hide-details
          />
          <v-btn
            class="ColourButton"
            small
            :style="{ background: profile[profileIdx].OC }"
            @click.stop="colourTemp = profile[profileIdx].OC; colourPick = 2; colourDialogue = true;"
          >
            {{ profile[profileIdx].OC }}
          </v-btn>
        </v-row>
        <v-row>
          <v-btn @click="modalMode = 1; modalNexus = true; addProfileNameString = 'Profile ' + profile.length;">
            Add Profile
          </v-btn>
          <v-btn @click="modalMode = 2; modalNexus = true">
            Remove Profile
          </v-btn>
          <v-btn @click="shiftProfileUp()">
            Shift Up
          </v-btn>
          <v-btn @click="shiftProfileDown()">
            Shift Down
          </v-btn>
        </v-row>
      </v-container>
    </div>

    <!---------   COLOUR MODAL --------->
    <v-dialog
      v-model="colourDialogue"
      max-width="300px"
      @click:outside.prevent="colourPickerClose();"
    >
      <v-card>
        <v-color-picker v-if="colourPick === 1" v-model="profile[profileIdx].CC" />
        <v-color-picker v-else-if="colourPick === 2" v-model="profile[profileIdx].OC" />
        <v-card-title :style="textStyle" style="font-weight:bold;">
          The quick brown fox jumps over the lazy dog
        </v-card-title>
        <v-card-actions>
          <v-btn @click="colourPickerClose();">
            Cancel
          </v-btn>

          <v-btn style="margin-left:auto" @click="colourPickerOK()">
            Ok
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!--========   COLOUR MODAL =======-->

    <!---------   NEXUS MODAL ---------
      1 Add profile
      2 Remove Profile
      3 Load Stream
      6 Login Check
    -->
    <v-dialog
      v-model="modalNexus"
      max-width="600px"
      persistent
      @click:outside="modalNexusOutsideClick();"
    >
      <!---------    ADD PROFILE     --------->
      <v-card v-if="modalMode === 1">
        <v-container>
          <v-card-title>
            Add New Profile.
          </v-card-title>
          <v-text-field
            v-model="addProfileNameString"
            label="Profile Name"
            placeholder="Profile Name"
            dense
            rounded
            outlined
          />
          <v-card-actions>
            <v-btn @click="modalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="addProfile()">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!---------    Remove PROFILE     --------->
      <v-card v-if="modalMode === 2">
        <v-container>
          <v-card-title>
            Confirm remove profile {{ profile[profileIdx].Name }}.
          </v-card-title>
          <v-card-actions>
            <v-btn @click="modalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="deleteProfile()">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!-------  LOAD VIDEO  ------->
      <v-card v-if="modalMode === 3">
        <v-container>
          <v-card-title>
            Load Video
          </v-card-title>
          <v-text-field v-model="activeURLInput" label="Stream Link" />
          <v-card-actions>
            <v-btn @click="modalNexus = false">
              Cancel
            </v-btn>

            <v-btn style="margin-left:auto" @click="loadChat(); modalNexus = false;">
              Ok
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>

      <!-------  LOGGIN IN CHECK ------->
      <v-card v-if="modalMode === 6">
        <v-container>
          <v-card-title>
            {{ loginStatusText }}
          </v-card-title>

          <v-card v-if="(loginStatus === 2) || (loginStatus === 3) || (loginStatus === 4)">
            <v-divider />
            <v-card-subtitle>
              {{ (loginNoteText === '') ? '' : 'Message from reviewer : ' + loginNoteText }}
            </v-card-subtitle>
            <v-text-field v-if="loginStatus != 4" v-model="applicationText" label="Note" />
            <v-divider />
          </v-card>

          <v-card-actions>
            <v-btn href="/">
              Back
            </v-btn>
            <v-btn v-if="loginStatus===-1" style="margin-left:auto" href="/login">
              Login
            </v-btn>

            <v-btn v-if="loginStatus===2" style="margin-left:auto" @click="applyTL();">
              Apply
            </v-btn>

            <v-btn v-if="loginStatus===3" style="margin-left:auto" @click="applyTL();">
              Re-apply
            </v-btn>
          </v-card-actions>
        </v-container>
      </v-card>
    </v-dialog>
    <!--========   NEXUS MODAL =======-->
  </v-container>
</template>

<script lang="ts">
import EnhancedEntry from "@/components/tlscripteditor/EnhancedEntry.vue";
import { TL_LANGS } from "@/utils/consts";
import { mdiPlusCircle, mdiMinusCircle, mdiCloseCircle } from "@mdi/js";
import { getVideoIDFromUrl } from "@/utils/functions";
import backendApi from "@/utils/backend-api";

import Vue from "vue";
import LoadScript from "vue-plugin-load-script";

Vue.use(LoadScript);

export default {
    name: "Tlscripteditor",
    metaInfo() {
        return {
            get title() {
                return "TLScriptEditor - Holodex";
            },
        };
    },
    components: {
        EnhancedEntry,
    },
    data() {
        return {
            TL_LANGS,
            mdiPlusCircle,
            mdiMinusCircle,
            mdiCloseCircle,
            menuBool: false,
            entries: [],
            profile: [{
                Name: "Default",
                Prefix: "",
                Suffix: "",
                useCC: false,
                CC: "#000000",
                useOC: false,
                OC: "#000000",
            }],
            profileContainer: {},
            profileIdx: 0,
            profileDisplay: false,
            profileDisplayTimer: undefined,
            inputString: "",
            // ------ COLOUR -------
            colourPick: 0,
            colourDialogue: false,
            colourTemp: "",
            // ------ MODAL --------
            modalNexus: true,
            modalMode: 6,
            addProfileNameString: "",
            // ------ SETTING ------
            TLLang: TL_LANGS[0],
            mainStreamLink: "",
            collabLinks: [""],
            // ---- ACTIVE VIDEO ----
            activeURLInput: "",
            activeURLStream: "",
            vidPlayer: false,
            vidIframeEle: null,
            player: null,
            IFOrigin: "",
            // ---- PRIVILIGE CHECK ----
            loginStatusText: "Checking login status...",
            loginStatus: 0, // 0: check login status, 1: check TL privilege, 2: not applied, 3: application rejected, 4: banned
            loginNoteText: "",
            applicationText: "",
            loggedInTL: true,
        };
    },
    computed: {
        textStyle() {
            return {
                "-webkit-text-fill-color": (this.profile[this.profileIdx].CC === "") ? "unset" : this.profile[this.profileIdx].CC,
                "-webkit-text-stroke-color": (this.profile[this.profileIdx].OC === "") ? "unset" : this.profile[this.profileIdx].OC,
                "-webkit-text-stroke-width": (this.profile[this.profileIdx].OC === "") ? "0px" : "1px",
            };
        },
        userdata() {
            return this.$store.state.userdata;
        },
        user() {
            return this.$store.state.userdata.user;
        },
    },
    mounted() {
        this.checkLoginValidity();
    },
    methods: {
        addEntry() {
            this.entries.push({
                Time: Date.now(),
                SText: this.profile[this.profileIdx].Prefix + this.inputString + this.profile[this.profileIdx].Suffix,
                CC: this.profile[this.profileIdx].useCC ? this.profile[this.profileIdx].CC : "",
                OC: this.profile[this.profileIdx].useOC ? this.profile[this.profileIdx].OC : "",
            });

            this.inputString = "";
        },
        deleteAuxLink(idx: number) {
            if (this.collabLinks.length !== 1) {
                this.collabLinks.splice(idx, 1);
            }
        },
        modalNexusOutsideClick() {
            if ((this.modalMode !== 3) && this.loggedInTL) {
                this.modalNexus = false;
            }
        },
        // ------------------------ PROFILE CONTROLLER ------------------------
        shiftProfileUp() {
            if (this.profileIdx > 1) {
                this.profileContainer = JSON.parse(JSON.stringify(this.profile[this.profileIdx - 1]));
                this.profile[this.profileIdx - 1] = this.profile[this.profileIdx];
                this.profile[this.profileIdx] = this.profileContainer;
                this.profileIdx -= 1;
                this.profileContainer = {};
            }
            this.showProfileList();
        },
        shiftProfileDown() {
            if ((this.profileIdx !== 0) && (this.profileIdx < this.profile.length - 1)) {
                this.profileContainer = JSON.parse(JSON.stringify(this.profile[this.profileIdx + 1]));
                this.profile[this.profileIdx + 1] = this.profile[this.profileIdx];
                this.profile[this.profileIdx] = this.profileContainer;
                this.profileIdx += 1;
                this.profileContainer = {};
            }
            this.showProfileList();
        },
        profileUp() {
            if (this.profileIdx === 0) {
                this.profileIdx = this.profile.length - 1;
            } else {
                this.profileIdx -= 1;
            }
            this.showProfileList();
        },
        profileDown() {
            if (this.profileIdx === this.profile.length - 1) {
                this.profileIdx = 0;
            } else {
                this.profileIdx += 1;
            }
            this.showProfileList();
        },
        profileJump(idx: number) {
            if (idx < this.profile.length) {
                this.profileIdx = idx;
            }
            this.showProfileList();
        },
        profileJumpToDefault() {
            this.profileIdx = 0;
            this.showProfileList();
        },
        addProfile() {
            if (this.addProfileNameString.trim() === "") {
                this.addProfileNameString = `Profile ${this.profile.length}`;
            }
            this.profile.push({
                Name: this.addProfileNameString,
                Prefix: "",
                Suffix: "",
                useCC: false,
                CC: "#000000",
                useOC: false,
                OC: "#000000",
            });
            this.profileIdx = this.profile.length - 1;
            this.modalNexus = false;
            this.showProfileList();
        },
        deleteProfile() {
            if (this.profileIdx !== 0) {
                this.profileIdx -= 1;
                this.profile.splice(this.profileIdx + 1, 1);
            }
            this.modalNexus = false;
            this.showProfileList();
        },
        showProfileList() {
            if (!this.profileDisplay) {
                this.profileDisplay = true;
            }

            if (this.profileDisplayTimer) {
                clearInterval(this.profileDisplayTimer);
            }

            this.profileDisplayTimer = setInterval(() => {
                this.profileDisplay = false;
                clearInterval(this.profileDisplayTimer);
            }, 3000);
        },
        colourPickerClose() {
            if (this.colourPick === 1) {
                this.profile[this.profileIdx].CC = this.colourTemp;
            } else if (this.colourPick === 2) {
                this.profile[this.profileIdx].OC = this.colourTemp;
            }
            this.colourDialogue = false;
        },
        colourPickerOK() {
            this.colourDialogue = false;
        },
        //= ======================== PROFILE CONTROLLER ========================

        // ---------------------- VIDEO CONTROLLER ----------------------
        loadChat() {
            this.activeURLStream = this.activeURLInput;
            this.vidPlayer = true;
            const StreamURL = getVideoIDFromUrl(this.activeURLStream);
            switch (StreamURL.type) {
                case "twitch":
                    this.LoadvideoTW(StreamURL.id, true);
                    break;

                case "twitch_vod":
                    this.LoadvideoTW(StreamURL.id, false);
                    break;

                case "twitcast":
                    this.SetupIframeTC(StreamURL.id, StreamURL.id, true);
                    break;

                case "twitcast_vod":
                    this.SetupIframeTC(StreamURL.id, StreamURL.channel.name, false);
                    break;

                case "niconico":
                    // niconico doesn't allow third party player hosting... at least for now...
                    // this.SetupIframeNC(StreamURL.id, true);
                    break;

                case "niconico_vod":
                    this.SetupIframeNC(StreamURL.id, false);
                    break;

                case "bilibili":
                    break;

                case "bilibili_vod":
                    this.SetupIframeBL(StreamURL.id);
                    break;

                default:
                    this.LoadvideoYT(StreamURL.id);
                    break;
            }
        },
        SetupIframeTC(MID: string, UID: string, Live: boolean): void {
            if (this.vidIframeEle) {
                this.vidIframeEle.parentNode?.removeChild(this.vidIframeEle);
            }
            this.vidIframeEle = document.createElement("iframe");
            if (Live) {
                this.vidIframeEle.src = `https://twitcasting.tv/${UID}/embeddedplayer/live?auto_play=false&default_mute=false`;
                this.vidIframeEle.loading = "lazy";
            } else {
                this.vidIframeEle.src = `https://twitcasting.tv/${UID}/embeddedplayer/${MID}?auto_play=false&default_mute=false`;
            }
            this.vidIframeEle.width = "100%";
            this.vidIframeEle.height = "100%";
            this.vidIframeEle.frameBorder = "0";

            this.LoadIframe("TC");
        },
        SetupIframeBL(VID: string): void {
            let embedID = "";
            if (this.vidIframeEle) {
                this.vidIframeEle.parentNode?.removeChild(this.vidIframeEle);
            }

            switch (VID.slice(0, 2).toLowerCase()) {
                case "bv":
                    embedID = `bvid=${VID.slice(2)}`;
                    break;

                case "av":
                    embedID = `aid=${VID.slice(2)}`;
                    break;

                default:
                    embedID = `cid=${VID}`;
                    break;
            }

            this.vidIframeEle = document.createElement("iframe");
            this.vidIframeEle.src = `https://player.bilibili.com/player.html?${embedID}&page=1&as_wide=1&high_quality=0&danmaku=0`;
            this.vidIframeEle.width = "100%";
            this.vidIframeEle.height = "100%";
            this.vidIframeEle.frameBorder = "0";

            this.LoadIframe("BL");
        },
        SetupIframeNC(VID: string, Live: boolean): void {
            if (this.vidIframeEle) {
                this.vidIframeEle.parentNode?.removeChild(this.vidIframeEle);
            }

            this.vidIframeEle = document.createElement("iframe");
            if (Live) {
                this.vidIframeEle.src = `https://live.nicovideo.jp/embed/${VID}`;
            } else {
                this.vidIframeEle.src = `https://embed.nicovideo.jp/watch/${VID}?autoplay=0`;
            }
            this.vidIframeEle.width = "100%";
            this.vidIframeEle.height = "100%";
            this.vidIframeEle.frameBorder = "0";
            this.vidIframeEle.allow = "encrypted-media;";

            this.LoadIframe("NC");
        },

        // -----------------  IFRAME  -----------------
        TimePing(timestamp: number): void {
            if (this.vidIframeEle?.contentWindow) {
                this.vidIframeEle?.contentWindow.postMessage({
                    n: "MChatXXMSync",
                    d: timestamp,
                }, this.IFOrigin);
            }
        },
        StartPing(): void {
            if (this.vidIframeEle?.contentWindow) {
                this.vidIframeEle?.contentWindow.postMessage({
                    n: "MChatXXMSync",
                    d: "s",
                }, this.IFOrigin);
            }
        },
        ModePing(Mode: string): void {
            switch (Mode) {
                case "TC":
                    this.IFOrigin = "https://twitcasting.tv";
                    break;

                case "NC":
                    this.IFOrigin = "https://embed.nicovideo.jp";
                    break;

                case "BL":
                    this.IFOrigin = "https://player.bilibili.com";
                    break;

                default:
                    this.IFOrigin = "";
                    break;
            }

            if (this.vidIframeEle?.contentWindow) {
                this.vidIframeEle?.contentWindow.postMessage({
                    n: "MChatXXMSync",
                    d: Mode,
                }, this.IFOrigin);
            }
        },
        PausePing(): void {
            if (this.vidIframeEle?.contentWindow) {
                this.vidIframeEle?.contentWindow.postMessage({
                    n: "MChatXXMSync",
                    d: "p",
                }, this.IFOrigin);
            }
        },
        SwitchPing(): void {
            if (this.vidIframeEle?.contentWindow) {
                this.vidIframeEle?.contentWindow.postMessage({
                    n: "MChatXXMSync",
                    d: "w",
                }, this.IFOrigin);
            }
        },
        LoadIframe(Mode: string): void {
            if (this.vidIframeEle) {
                const PlayerDiv = document.getElementById("player");
                if (PlayerDiv) {
                    PlayerDiv.append(this.vidIframeEle);

                    this.vidIframeEle.onload = () => {
                        this.ModePing(Mode);
                    };

                    window.addEventListener("message", (e:any) => {
                        this.SessionStorageListener(e);
                    });
                }
            }
        },
        SessionStorageListener(e: any):void {
            if (e.origin === this.IFOrigin) {
                if (e.data.n === "MSyncXMChatX") {
                    switch (e.data.d) {
                        case "p":
                            // this.PauseTracker = true;
                            break;

                        case "s":
                            // this.PauseTracker = false;
                            break;

                        default:
                            if (typeof e.data.d === "number") {
                                /*
                    this.TimerTime = e.data.d;
                    this.ScrollCalculator();
                    */
                            }
                            break;
                    }
                }
            }
        },
        //= ================  IFRAME  =================

        // -----------------  YT  -----------------
        LoadvideoYT(VID: string) {
            if (window.YT) {
                this.startVideoYT(VID);
                return;
            }

            const tag = document.createElement("script");
            tag.src = "https://www.youtube.com/iframe_api";
            const firstScriptTag = document.getElementsByTagName("script")[0];
            if (firstScriptTag.parentNode) {
                firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
            }

            window.onYouTubeIframeAPIReady = () => this.startVideoYT(VID);
        },
        startVideoYT(VID: string) {
            this.player = new window.YT.Player("player", {
                videoId: VID,
                playerVars: {
                    playsinline: 1,
                },
                events: {
                    onReady: this.ReadyStateYT.bind(this),
                    onStateChange: this.onPlayerStateChangeYT.bind(this),
                },
            });
        },
        onPlayerStateChangeYT() {
            // this.TogglePlayPauseYT();
        },
        ReadyStateYT() {
            // this.PauseTracker = false;
        },
        TogglePlayPauseYT() {
            /*
          switch (this.player.getPlayerState()) {
            case 1:
              break;

            case 2:
              this.StopTimer(false);
              break;

            case 3:
              break;
          }
          */
        },
        StartTrackerYT(): void {
            /*
          this.PauseTracker = true;
          if (this.TrackerDelegate) {
            clearInterval(this.TrackerDelegate);
            this.TrackerDelegate = undefined;
          }

            this.TrackerDelegate = setInterval(() => {
            if (!this.PauseTracker) {
              this.TimerTime = this.player.getCurrentTime()*1000;
            }
            this.ScrollCalculator();
          }, 20);
          */
        },
        //= ================  YT  =================

        // -----------------  TW  -----------------
        LoadvideoTW(VID:string, Live: boolean) {
            if (window.Twitch) {
                this.startVideoTW(VID, Live);
                return;
            }

            const tag = document.createElement("script");
            tag.src = "https://player.twitch.tv/js/embed/v1.js";
            const firstScriptTag = document.getElementsByTagName("script")[0];
            if (firstScriptTag.parentNode) {
                firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
            }

            const Checker = setInterval(() => {
                if (window.Twitch) {
                    clearInterval(Checker);
                    this.startVideoTW(VID, Live);
                }
            }, 1000);
        },
        startVideoTW(VID: string, Live: boolean) {
            if (Live) {
                this.player = new window.Twitch.Player("player", {
                    width: "100%",
                    height: "100%",
                    channel: VID,
                    autoplay: false,
                    time: "0h0m0s",
                });
            } else {
                this.player = new window.Twitch.Player("player", {
                    width: "100%",
                    height: "100%",
                    video: VID,
                    autoplay: false,
                    time: "0h0m0s",
                });
            }

            this.player.addEventListener(window.Twitch.Player.PAUSE, () => {
            // this.PauseTracker = true;
            });

            this.player.addEventListener(window.Twitch.Player.PLAY, () => {
            // this.PauseTracker = false;
            });

            this.player.addEventListener(window.Twitch.Player.SEEK, (e: any) => {
                this.TimerTime = e.position * 1000;
            /*
            this.ScrollCalculator();
            */
            });
        },
        StartTrackerTW(): void {
            /*
          this.PauseTracker = true;
          if (this.TrackerDelegate) {
            clearInterval(this.TrackerDelegate);
            this.TrackerDelegate = undefined;
          }

          this.TimeSaddle = Date.now();
          this.TrackerDelegate = setInterval(() => {
            if ((Date.now() - this.TimeSaddle < 1000) && (!this.PauseTracker)) {
              this.TimerTime += Date.now() - this.TimeSaddle;
            }
            this.TimeSaddle = Date.now();
            this.ScrollCalculator();
          }, 20);
          */
        },
        //= ================  TW  =================

        //= ====================== VIDEO CONTROLLER ======================

        // ------------------------ LOGIN STATUS CHECK ------------------------
        async checkLoginValidity() {
            const check = await backendApi.loginIsValid(this.userdata.jwt);
            if (check === false) {
                this.$store.dispatch("logout");
                this.loginStatus = -1;
                this.loginStatusText = "User not logged in.";
            } else if (check.data && check.data.id) {
                this.$store.commit("setUser", { user: check.data, jwt: this.userdata.jwt });
                this.loginStatus = 1;
                this.loginStatusText = "Logged in, checking TL privilege...";
                this.checkUserTLStatus();
            }
        },
        async checkUserTLStatus() {
            backendApi.checkTLStatus({
                id: this.user.id,
            }).then(({ status, data }) => {
                if (status === 200) {
                    switch (data.status) {
                        case "OK":
                            this.loginStatusText = "OK!";
                            this.modalMode = 3;
                            return;

                        case "BANNED":
                            this.loginStatusText = "Banned.";
                            this.loginNoteText = data.note;
                            this.loginStatus = 4;
                            return;

                        case "REJECTED":
                            this.loginStatusText = "Application rejected.";
                            this.loginNoteText = data.note;
                            this.applicationText = "Add note here to help review your application, it can be your qualification or link to past stream that you've translated.";
                            this.loginStatus = 3;
                            return;

                        case "APPLIED":
                            this.loginStatusText = "Application has been sent, still waiting for review.";
                            this.loginStatus = -2;
                            return;

                        case "NOT":
                            this.loginStatusText = "Not applied for TL role.";
                            this.applicationText = "Add note here to help review your application, it can be your qualification or link to past stream that you've translated.";
                            this.loginStatus = 2;
                            return;

                        default:
                            this.loginStatus = -2;
                            this.loginStatusText = "Can't reach server";
                    }
                } else {
                    this.loginStatus = -2;
                    this.loginStatusText = "Can't reach server";
                }
            }).catch((err) => {
                this.loginStatus = -2;
                this.loginStatusText = "Can't reach server";
                console.log(err);
            });
        },
        applyTL() {
            backendApi.applyTL({
                id: this.user.id,
                applynote: this.applicationText,
            }).then(({ status }) => {
                if (status === 200) {
                    this.loginStatusText = "Application sent! Please be patient while we check your application...";
                    this.loginStatus = -2;
                } else {
                    this.loginStatus = -2;
                    this.loginStatusText = "Can't reach server";
                }
            }).catch((err) => {
                this.loginStatus = -2;
                this.loginStatusText = "Can't reach server";
                console.log(err);
            });
        },
        //= ======================= LOGIN STATUS CHECK ========================
    },
};
</script>

<style>
.TopMenu {
  width:100%;
  position:absolute;
  top:0px;
  left:0px;
  z-index: 1;
}
.ChatOuterContainer{
  overflow-y: auto;
}
.ChatInnerContainer{
  position: absolute;
  bottom: 0px;
  width: 100%;
}
.ColourButton {
  margin-top: 19px;
  margin-left: 5px;
}
.ProfileListCard {
  position: absolute;
  bottom: 5px;
  right: 5px;
}
.ChatPanelContainer{
  display: grid;
  grid-auto-flow: column;
}
</style>
