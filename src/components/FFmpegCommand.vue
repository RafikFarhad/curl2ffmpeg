<template>
  <div>
    <div class="card bg-secondary text-white">
      <div class="card-header">
        <div class="d-flex align-items-center">
          <div class="">Result</div>
          <div class="ms-auto m-n1">
            <button class="btn btn-sm btn-outline-light" v-on:click="copy" :disabled="coppyButtonDisabled">
              {{ copyButtonText }}
            </button>
          </div>
        </div>
      </div>
      <div class="card-body">
        <div ref="command" class="monospace">
          [ -f '{{ esc(filename) }}' ] ||
          ffmpeg
          -hide_banner
          -loglevel {{ logLevel }}
          -stats
          -stats_period 1
          -n
          <template v-if="robustDownload">-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5 -rw_timeout 15000000 </template>
          <span v-if="headers.length > 0">-headers </span>
          <span v-for="(header, index) in headers" :key="index">'{{ esc(header) }}'$'\r\n'</span>
          -i '{{ esc(url) }}'
          -c copy
          '{{ esc(filename) }}'
          <template v-if="convert720">
            &amp;&amp; echo "done"
            &amp;&amp; height=$(ffprobe -v error -select_streams v:0 -show_entries stream=height -of csv=p=0 '{{ esc(filename) }}')
            &amp;&amp; if [ "$height" -gt 720 ]; then
            ffmpeg
            -hide_banner
            -loglevel {{ logLevel }}
            -stats
            -stats_period 1
            -y
            -i '{{ esc(filename) }}'
            -vf scale=-2:720,format=yuv420p
            <template v-if="videoToolbox">-c:v h264_videotoolbox -b:v 2500k -tag:v avc1</template>
            <template v-else>-c:v libx264 -preset veryfast -crf 28</template>
            -c:a aac
            -b:a 128k
            '{{ esc(filename720) }}'<template v-if="verify"> &amp;&amp; echo "verifying re-encode against source..." &amp;&amp; ffmpeg -hide_banner -i '{{ esc(filename) }}' -i '{{ esc(filename720) }}' -filter_complex "[0:v]scale=-2:720[ref];[ref][1:v]ssim" -f null -</template>;
            else echo "Source height ${height}p is not greater than 720, skipping 720p conversion"; fi
          </template>
          <template v-if="notify">
            &amp;&amp; printf '\a'
            &amp;&amp; osascript -e 'display notification "ffmpeg finished" with title "curl2ffmpeg"'
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  props: {
    url: {
      type: String,
      required: true
    },
    headers: {
      type: Array,
      default: () => []
    },
    filename: {
      type: String,
      required: true
    },
    convert720: {
      type: Boolean,
      default: true
    },
    verbose: {
      type: Boolean,
      default: false
    },
    videoToolbox: {
      type: Boolean,
      default: true
    },
    robustDownload: {
      type: Boolean,
      default: true
    },
    notify: {
      type: Boolean,
      default: false
    },
    verify: {
      type: Boolean,
      default: false
    }
  },
  setup(props) {
    const copyButtonText = ref('Copy');
    const coppyButtonDisabled = ref(false);
    const command = ref(null);

    const copy = async () => {
      await navigator.clipboard.writeText(command.value.textContent.trim());
      copyButtonText.value = '✓ Copied';
      coppyButtonDisabled.value = true;
      setTimeout(() => {
        copyButtonText.value = 'Copy';
        coppyButtonDisabled.value = false;
      }, 500);
    };

    const esc = (value) => String(value).replace(/'/g, "'\\''");

    return {
      copyButtonText,
      coppyButtonDisabled,
      command,
      copy,
      esc,
      headers: computed(() => props.headers),
      url: computed(() => props.url),
      filename: computed(() => props.filename),
      convert720: computed(() => props.convert720),
      videoToolbox: computed(() => props.videoToolbox),
      robustDownload: computed(() => props.robustDownload),
      notify: computed(() => props.notify),
      verify: computed(() => props.verify),
      logLevel: computed(() => (props.verbose ? 'verbose' : 'warning')),
      filename720: computed(() => {
        const dot = props.filename.lastIndexOf('.');
        if (dot <= 0) {
          return `${props.filename}_720`;
        }
        return `${props.filename.slice(0, dot)}_720${props.filename.slice(dot)}`;
      })
    };
  }
};
</script>
