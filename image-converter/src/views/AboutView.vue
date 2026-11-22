<template>
  <div class="image-converter">
    <input type="file" @change="handleFileUpload" />
    <div v-if="imageSrc">
      <p>Converted Image:</p>
      <img :src="imageSrc" alt="Converted Image" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import heic2any from 'heic2any';

export default defineComponent({
  name: 'ImageConverter',
  setup() {
    const imageSrc = ref<string | null>(null);

    const handleFileUpload = async (event: Event) => {
      const input = event.target as HTMLInputElement;
      if (input.files && input.files[0]) {
        const file = input.files[0];

        // Check if the file is in HEIC format based on MIME type
        if (file.type === 'image/heic' || file.type === 'image/heif') {
          try {
            // Convert HEIC to JPEG
            const convertedBlob = await heic2any({ blob: file, toType: 'image/jpeg' });

            // Create a URL for the converted image and set it as imageSrc
            const convertedUrl = URL.createObjectURL(convertedBlob);
            imageSrc.value = convertedUrl;

            // Release the object URL after use (optional)
            URL.revokeObjectURL(convertedUrl);

          } catch (error) {
            console.error('Error converting image:', error);
            alert('An error occurred during conversion.');
          }
        } else {
          alert('Please upload a HEIC image.');
        }
      } else {
        alert('No file selected.');
      }
    };

    return { imageSrc, handleFileUpload };
  }
});

fetch("https://alexcorvi.github.io/heic2any/demo/14.heic")
  .then((res) => res.blob())
  .then((blob) => heic2any({
    blob,
    toType: "image/jpeg",
    quality: 0.3,
    multiple: true
  }))
  .then((conversionResult) => {
    document.getElementById("target").innerHTML = '';
    conversionResult.forEach(image => {
      var url = URL.createObjectURL(image);
      document.getElementById("target").innerHTML = document.getElementById("target").innerHTML + `<a target="_blank" href="${url}"><img src="${url}"></a>`;
    });
  })
  .catch((e) => {
    console.log(e);
  });
</script>

<style>
.image-converter {
  text-align: center;
  margin-top: 20px;
}

img {
  max-width: 100%;
  height: auto;
}
</style>
