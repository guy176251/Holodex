<template>
  <v-card
    class="text-body-2 tl-overlay"
    tile
    flat
    style="width: 100%"
  >
    <v-card-subtitle class="py-1 d-flex justify-space-between">
      <div>TLdex [{{ liveTlLang }}]</div>
      <span>
        <v-btn
          icon
          x-small
          class="mr-1"
          title="-2s"
          @click="timeOffset -= 2000"
        >
          <v-icon>
            {{ mdiTransferLeft }}
          </v-icon>
        </v-btn>
        <code class="mr-1">{{ `${timeOffset >= 0 ? "+" : ""}${timeOffset / 1000}s` }}</code>
        <v-btn
          icon
          x-small
          class="mr-1"
          title="+2s"
          @click="timeOffset += 2000"
        >
          <v-icon>
            {{ mdiTransferRight }}
          </v-icon>
        </v-btn>
        <v-btn
          icon
          x-small
          class="mr-1"
          :title="$t('views.watch.chat.showSubtitle')"
          @click="showSubtitle = !showSubtitle"
        >
          <v-icon :color="showSubtitle ? 'primary' :''">
            {{ mdiSubtitlesOutline }}
          </v-icon>
        </v-btn>
        <v-dialog v-model="expanded" width="800">
          <template #activator="{ on, attrs }">
            <v-btn
              icon
              x-small
              v-bind="attrs"
              :title="$t('views.watch.chat.expandTL')"
              v-on="on"
            >
              <v-icon>
                {{ mdiArrowExpand }}
              </v-icon>
            </v-btn>
          </template>
          <v-card>
            <portal-target name="expandedMessage" class="d-flex tl-expanded" />
            <v-divider />
            <v-card-actions>
              <v-spacer />
              <v-btn text color="red" @click="expanded = false">{{ $t("views.app.close_btn") }}</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <WatchLiveTranslationsSetting />
      </span>
    </v-card-subtitle>
    <v-divider />
    <portal to="expandedMessage" :disabled="!expanded" slim>
      <virtual-list
        ref="tlBody"
        class="archive tl-body px-1 py-0 px-lg-3"
        :style="{
          'font-size': liveTlFontSize + 'px',
        }"
        :data-component="ChatMessage"
        :data-key="getKey"
        :data-sources="dividedTLs"
        :item-height="20"
        :item-class-add="activeClass"
        :keeps="50"
        @click.native="handleClick"
      />
    </portal>
    <portal v-if="showSubtitle" :to="`${video.id}-overlay`">
      <WatchSubtitleOverlay :messages="toDisplay" />
    </portal>
  </v-card>
</template>

<script lang="ts">
import VirtualList from "vue-virtual-scroll-list";
import { mdiTransferRight, mdiTransferLeft } from "@mdi/js";
import WatchLiveTranslationsSetting from "./LiveTranslationsSetting.vue";
import ChatMessage from "./ChatMessage.vue";
import chatMixin from "./chatMixin";
import WatchSubtitleOverlay from "../watch/WatchSubtitleOverlay.vue";

export default {
    name: "ArchiveTranslations",
    components: {
        WatchLiveTranslationsSetting,
        VirtualList,
        WatchSubtitleOverlay,
    },
    mixins: [chatMixin],
    data() {
        return {
            ChatMessage,
            curIndex: 0,
            timeOffset: 0, // for offsetting archive TL
            mdiTransferRight,
            mdiTransferLeft };
    },
    computed: {
        dividedTLs() {
            const self = this;
            const filtered = this.tlHistory.filter((m) => !this.blockedNames.has(m.name));
            return filtered.map((item, index, arr) => {
                const shouldHideAuthor = index > 0 && (!(index === 0
                    || index === arr.length - 1
                    || item.name !== arr[index - 1].name
                    || !!item.breakpoint));
                // timestamp is in milliseconds.
                const newtime = item.timestamp + self.timeOffset;
                const relativeMs = item.relativeMs + self.timeOffset;
                return { ...item, shouldHideAuthor, relativeMs, timestamp: newtime };
            });
        },
        toDisplay() {
            if (!this.dividedTLs.length || !this.showSubtitle) return [];
            const startIdx = Math.max(this.curIndex - 1, 0);
            // Grab previous and current message
            const buffer = this.dividedTLs.slice(startIdx, startIdx + 2);
            return buffer.filter((m) => {
                const displayTime = +m.duration || (m.message.length * 65 + 1800);
                return this.currentTime * 1000 >= m.relativeMs && this.currentTime * 1000 < m.relativeMs + displayTime;
            });
        },
    },
    watch: {
        liveTlLang() {
            this.loadMessages(true, true);
        },
        currentTime(time) {
            if (!this.dividedTLs.length) return;
            const msTime = time * 1000;
            const cur = this.dividedTLs[this.curIndex].relativeMs;
            // time jumped forward too fast, or backwards. Exhaustive search for next spot

            const startIndex = time < cur ? 0 : this.curIndex;
            for (let i = startIndex; i < this.tlHistory.length; i += 1) {
                if (i === this.dividedTLs.length - 1) {
                    this.curIndex = this.dividedTLs.length - 1;
                    return;
                }
                if (msTime <= this.dividedTLs[i].relativeMs) {
                    this.curIndex = Math.max(i - 1, 0);
                    return;
                }
            }
        },
        curIndex(idx) {
            this.scrollToIndex(idx);
        },
    },
    created() {
        this.loadMessages(true, true);
    },
    methods: {
        activeClass(index) {
            return index === this.curIndex ? "active-message" : "";
        },
        getKey(item) {
            return item.timestamp + item.message + item.name;
        },
        handleClick(e) {
            if (e.target.matches(".tl-message, .tl-message *")) {
                this.$emit("timeJump", +e.target.parentElement.getAttribute("data-time"), true, true);
                e.preventDefault();
            }
        },

        scrollToIndex(idx) {
            const ref = this.$refs.tlBody;
            const idxSize = ref.virtual.sizes.get(idx) ?? 50;
            const idxOffset = ref.virtual.getOffset(idx);
            const nearBottom = idxOffset + ref.getClientSize() > ref.getScrollSize();
            if (nearBottom) {
                ref.scrollToBottom();
            } else {
                ref.scrollToOffset(idxOffset - (ref.getClientSize() / 2) + idxSize);
            }
        },
    },
};
</script>

<style>
.tl-body.archive {
    overflow-y: auto;
    position: relative;
    overscroll-behavior: contain;
    height: calc(100% - 32px);
    display: flex;
    flex-direction: column-reverse;
    flex-direction: column;
    line-height: 1.35;
    letter-spacing: 0.0178571429em !important;
}

.active-message {
    position: relative;
}
.active-message {
    z-index: 0;
}
.active-message .tl-message::before {
    content: "";
    background-color: var(--v-primary-base);
    opacity: 0.25;
    width: calc(100%);
    height: calc(100%);
    background-size: cover;
    position: absolute;
    top: -1px;
    left: 0;
    z-index: -1;
}
</style>
