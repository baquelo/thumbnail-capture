<script lang="ts" setup>
import axios from "axios";
import html2canvas from "html2canvas";
import { ref } from "vue";

const thumbUrl = ref(""); // initialize the thumbUrl to an empty string

const handleFileUpload = async (e) => {
  const file = e.target.files[0];
  const formData = new FormData();
  formData.append("file", file);

  try {
    // Upload to server
    // await axios.post(`/api/courses/store-thumbnail`, formData, {
    //   headers: {
    //     "Content-Type": "multipart/form-data",
    //   },
    // });

    // set thumbUrl to the uploaded file
    thumbUrl.value = URL.createObjectURL(file);

    // clear the input
    e.target.value = null;
  } catch (error) {
    console.log("Error uploading file: ", error);
  }
};

const createThumbnail = async () => {
  try {
    // Code to capture styles of the elements we need
    const titleStyle = window.getComputedStyle(
      document.querySelector("#page-title")
    );
    const backgroundColor = window.getComputedStyle(
      document.body
    ).backgroundColor;

    // Create temporary div
    const tmpDiv = document.createElement("div");
    tmpDiv.style.cssText = `
            width: 800px;
            height: 600px;
            font-family: ${titleStyle.fontFamily};
            color: ${titleStyle.color};
            background-color: ${backgroundColor};
            border: 1px solid black;
        `;

    // Append to the body
    document.body.appendChild(tmpDiv);

    // Capture the screenshot
    const options = {
      allowTaint: true,
      useCORS: true,
      height: 600,
      width: 800,
    };
    // const canvas = await html2canvas(tmpDiv, options);
    const canvas = await html2canvas(document.querySelector("#app"), options);

    // Convert the canvas to a data URL
    const imageDataUrl = canvas.toDataURL("image/jpeg", 0.95);

    // Convert DataURL to Blob
    const response = await fetch(imageDataUrl);
    const blob = await response.blob();
    thumbUrl.value = imageDataUrl;

    // Create FormData
    const formData = new FormData();
    formData.append("file", blob, "screenshot.jpg");

    // Upload to server
    // await axios.post(`/api/courses/store-thumbnail`, formData, {
    //   headers: {
    //     "Content-Type": "multipart/form-data",
    //   },
    // });

    // Remove the temporary div
    document.body.removeChild(tmpDiv);
  } catch (error) {
    console.log("Error creating thumbnail: ", error);
  }
};
</script>

<template>
  <!-- button to capture screenshot -->
  <div class="d-flex row gap-1">
    <button class="btn btn-primary" @click="createThumbnail">
      Create Thumbnail
    </button>
    <!-- BS5 image to display thumbnail -->
    <img :src="thumbUrl" alt="thumb" class="img-thumbnail" />
    <div class="mb-3">
      <label for="formFile" class="form-label">Or, Upload your own</label>
      <input
        type="file"
        class="form-control"
        id="formThumbnail"
        accept="image/*"
        @change="handleFileUpload"
      />
    </div>
  </div>
</template>
