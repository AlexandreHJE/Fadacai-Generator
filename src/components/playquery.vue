<template>
  <b-card title="已排序聲音" sub-title="點按按鈕可以移除已排序的聲音">
    <hr />
    <p v-show="snds.length == 0" style="color: #ccc">請先在下方選擇音檔</p>
    <draggable
      v-model="snds"
      @start="dragging = true"
      @end="dragging = false"
      v-bind="dragOptions"
    >
      <transition-group>
        <b-button
          class="m-1"
          v-for="(s, idx) in snds"
          :key="idx + 0"
          variant="success"
          @click="remove(idx)"
          >{{ s.name }}</b-button
        >
      </transition-group>
    </draggable>
    <hr />
    <b-button class="m-1" variant="primary" @click="play()" ref="btnPlay"
      >播放</b-button
    >
    <b-button class="m-1" variant="warning" @click="stop()" ref="btnStop"
      >停止</b-button
    >
    <b-button class="m-1" variant="success" @click="getUrl()">匯出</b-button>
    <b-button class="m-1" variant="info" @click="downloadFile()">下載</b-button>
    <b-button class="m-1" variant="danger" @click="removeAll()">清空</b-button>
  </b-card>
</template>
<script>
import Crunker from "crunker";
import { eventBus } from "../main.js";
import draggable from "vuedraggable";
let nowPlaying = -1;
let isPlaying = false;
export default {
  name: "query",
  components: {
    draggable
  },
  props: {
    sounds: Array,
    query: String,
    sids: String
  },
  data() {
    let snds = [];
    return {
      snds
    };
  },
  methods: {
    open: function(which, e) {
      // Prevents clicking the link from doing anything
      e.preventDefault();
    },
    play() {
      if (this.snds.length > 0) {
        isPlaying = true;
        nowPlaying = 0;
        this.$refs.btnPlay.setAttribute("disabled", "true");
        let audio = new Audio(this.snds[nowPlaying].file);
        audio.play();
        audio.onended = () => {
          nowPlaying++;
          if (nowPlaying < this.snds.length && isPlaying) {
            audio.src = this.snds[nowPlaying].file;
            audio.play();
          } else {
            nowPlaying = -1;
            isPlaying = false;
            this.$refs.btnPlay.removeAttribute("disabled");
          }
        };
      }
    },
    stop() {
      nowPlaying = -1;
      isPlaying = false;
      this.$refs.btnPlay.removeAttribute("disabled");
    },
    getUrl() {
      let url = "";
      for (let i = 0; i < this.snds.length; i++) {
        url = url + this.snds[i].id + ",";
      }
      url = url.substr(0, url.length - 1);
      this.$router.push({
        name: "home",
        query: { sids: url }
      });
      alert("匯出成功，可使用目前網址分享你的創作");
    },
    downloadFile() {
      const selectedSounds = this.snds.map(snd => snd.file);
      const audio = new Crunker();
      audio
        .fetchAudio(...selectedSounds)
        .then(buffers => audio.concatAudio(buffers))
        .then(merged => audio.export(merged, "audio/mp3"))
        .then(output => audio.download(output.blob, "fadacai"))
        .catch(error => {
          throw new Error(error);
        });
    },
    remove(idx) {
      if (nowPlaying == -1) {
        this.snds.splice(idx, 1);
      }
    },
    removeAll() {
      this.snds = [];
    }
  },
  mounted() {
    // when soundbtns.vue btn clicked
    eventBus.$on("addSound", message => {
      let isIn = -1;
      for (let i = 0; i < this.sounds.length; i++) {
        if (message == this.sounds[i].id) {
          isIn = i;
          break;
        }
      }
      if (isIn > -1) {
        this.snds.push({
          file: this.sounds[isIn].file,
          name: this.sounds[isIn].name,
          id: this.sounds[isIn].id
        });
      }
    });
  },
  computed: {
    dragOptions() {
      return {
        animation: 200,
        group: "description",
        disabled: false,
        ghostClass: "ghost"
      };
    }
  },
  watch: {
    // use watch because ajax sound list has delay
    // load url when sound list is ready
    sounds: function(newVal) {
      if (this.sids != undefined && this.sids.length > 0) {
        let querys = this.sids.split(",");
        if (querys.length > 0) {
          // loop all url query
          for (let i = 0; i < querys.length; i++) {
            // find it in sound list
            let isIn = -1;
            for (let j = 0; j < newVal.length; j++) {
              let id = newVal[j].id;
              if (id == querys[i]) {
                isIn = j;
                break;
              }
            }
            if (isIn > -1) {
              this.snds.push({
                file: newVal[isIn].file,
                name: newVal[isIn].name,
                id: newVal[isIn].id
              });
            }
          }
        }
      } else if (this.query != undefined && this.query.length > 0) {
        let querys = this.query.split(",");
        if (querys.length > 0) {
          for (let i = 0; i < querys.length; i++) {
            let isIn = -1;
            for (let j = 0; i < newVal.length; j++) {
              let name = newVal[j].name;
              if (name == querys[i]) {
                isIn = j;
                break;
              }
            }
            if (isIn > -1) {
              this.snds.push({
                file: newVal[isIn].file,
                name: newVal[isIn].name,
                id: newVal[isIn].id
              });
            }
          }
        }
      }
    }
  }
};
</script>
