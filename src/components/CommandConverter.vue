<template>
  <div>
    <div class="card my-4">
      <div class="card-body">
        <form acton="#" @submit.prevent="submit">
          <div class="mb-3">
            <label for="command" class="form-label">cURL command</label>
            <textarea v-model="command" id="command" class="form-control monospace" rows="10" />
          </div>
            <label for="command" class="form-label">Output file name</label>
          <div class="mb-3">
            <input v-model="filename" id="filename" type="text" class="form-control" />
          </div>
          <div class="mb-3">
            <div class="form-check">
              <input v-model="convert720" id="convert720" type="checkbox" class="form-check-input" />
              <label for="convert720" class="form-check-label">Convert to 720p when source height is greater than 720</label>
            </div>
            <div class="form-check">
              <input v-model="verboseLogging" id="verboseLogging" type="checkbox" class="form-check-input" />
              <label for="verboseLogging" class="form-check-label">Verbose logging</label>
            </div>
            <div class="form-check">
              <input v-model="videoToolbox" id="videoToolbox" type="checkbox" class="form-check-input" />
              <label for="videoToolbox" class="form-check-label">Use VideoToolbox hardware encoder (macOS)</label>
            </div>
            <div class="form-check">
              <input v-model="robustDownload" id="robustDownload" type="checkbox" class="form-check-input" />
              <label for="robustDownload" class="form-check-label">Robust download (auto-reconnect)</label>
            </div>
            <div class="form-check">
              <input v-model="notifyOnFinish" id="notifyOnFinish" type="checkbox" class="form-check-input" />
              <label for="notifyOnFinish" class="form-check-label">Notify on finish (bell + macOS notification)</label>
            </div>
            <div class="form-check">
              <input v-model="verify" id="verify" type="checkbox" class="form-check-input" />
              <label for="verify" class="form-check-label">Verify re-encode matches source (SSIM)</label>
            </div>
          </div>
          <button type="submit" class="btn btn-primary">Convert</button>
        </form>
      </div>
    </div>

    <div v-if="url">
      <FFMpegCommand :url="url" :headers="headers" :filename="filename" :convert720="convert720" :verbose="verboseLogging" :video-toolbox="videoToolbox" :robust-download="robustDownload" :notify="notifyOnFinish" :verify="verify" />
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { uniqueNamesGenerator, adjectives, animals } from 'unique-names-generator';
import CurlCommand from '~/lib/CurlCommand.js';
import FFMpegCommand from '~/components/FFmpegCommand.vue';

const randomName = () => uniqueNamesGenerator({
  dictionaries: [adjectives, animals],
  separator: '_',
  length: 2
});

export default {
  components: {
    FFMpegCommand
  },
  setup() {
    const command = ref('');
    const url = ref('');
    const headers = ref([]);
    const filename = ref(`${randomName()}.mp4`);
    const convert720 = ref(true);
    const verboseLogging = ref(false);
    const videoToolbox = ref(true);
    const robustDownload = ref(true);
    const notifyOnFinish = ref(false);
    const verify = ref(false);

    const submit = () => {
      url.value = '';
      headers.value = '';
      if (!command.value) {
        return;
      }
      const curlCommand = new CurlCommand(command.value.trim());
      url.value = curlCommand.url;
      headers.value = curlCommand.headers;
    };

    return {
      command,
      url,
      headers,
      filename,
      convert720,
      verboseLogging,
      videoToolbox,
      robustDownload,
      notifyOnFinish,
      verify,
      submit
    };
  }
};
</script>

<style>
.monospace {
  font-family: monospace;
}
</style>
