<template>
  <v-card>
    <v-divider></v-divider>
    <v-form>
      <v-switch inset label="深色模式" v-model="dark_toggle"></v-switch>
    </v-form>
    <v-btn @click="switchfont">Switch font</v-btn>
    <v-row style="margin: auto">
      <slider :attrb="imageBattr" @input="handleUpdate"></slider>

      <v-col style="margin-left: 20px">
        <v-list>
          <v-chip> RESULT </v-chip>
          <v-img
            :src="imageUrl1"
            alt="Fetched Image"
            height="400px"
            width="400px"
          />
        </v-list>
        <v-list>
          <v-chip> IMAGE A </v-chip>
          <v-img
            :src="imageUrl2"
            alt="Fetched Image"
            height="400px"
            width="400px"
          />
          <v-select
            label="Result"
            :items="listOfFonts"
            v-model="imageB"
          ></v-select>
          <v-divider></v-divider>
          <v-select
            label="ImageA"
            :items="listOfFonts"
            v-model="imageA"
          ></v-select>
        </v-list>
      </v-col>
    </v-row>
  </v-card>
</template>
<script lang="ts" setup>
import { useTheme } from "vuetify/lib/framework.mjs";
import slider from "@/components/slider.vue";
import { watch, ref, onMounted } from "vue";
const imageUrl1 = ref("");
const imageUrl2 = ref("");
const listOfFonts = ref([]);
const imageB = ref("");
const imageA = ref("");
const imageBattr = ref("");
const theme = useTheme();
const dark_toggle = ref(theme.global.name.value === "dark");
function apply_theme() {
  var theme_name = "";
  if (dark_toggle.value) {
    theme_name = "dark";
  } else {
    theme_name = "light";
  }
  theme.global.name.value = theme_name;
  localStorage.setItem("theme", theme_name);
}
apply_theme();
watch(dark_toggle, apply_theme);
async function change_attr(filename: string = "", data: object) {
  const response = await fetch(
    `http://127.0.0.1:5000/api/attributes/${filename}`,
    {
      method: "POST",
      headers: {
        Accept: "application/json",
        "Content-Type": "application/json",
      },
      body: JSON.stringify(data),
    }
  );
  return response;
}
async function fetch_api(path: string = "") {
  const response = await fetch(`http://127.0.0.1:5000/api/image/${path}`, {
    method: "GET",
  });
  const blob = await response.blob();
  return URL.createObjectURL(blob);
}
async function run_test() {
  const response = await fetch(`http://127.0.0.1:5000`, {
    method: "GET",
  });
  return response;
}
async function fetch_list() {
  try {
    const response = await fetch(`http://127.0.0.1:5000/api/list`, {
      method: "GET",
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching data:", error);
    return null; // or handle the error appropriately
  }
}
async function fetch_imageB() {
  try {
    const response = await fetch(`http://127.0.0.1:5000/api/image/B `, {
      method: "GET",
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching data:", error);
    return null; // or handle the error appropriately
  }
}
async function fontlist() {
  try {
    const res = await fetch_list();
    if (res) {
      const data = res.data; // assuming your response structure has a 'data' field
      // Process your data here as needed
      listOfFonts.value = data;
      console.log(listOfFonts.value);
    } else {
      console.log("No data received");
    }
  } catch (error) {
    console.error("Error processing the font list:", error);
  }
}
//create a post for'/api/image'
async function give_image() {
  const response = await fetch(`http://127.0.0.1:5000/api/image`, {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      imageA: imageA.value,
      imageB: imageB.value,
    }),
  });
}
async function switchfont() {
  await give_image();
  getimageB();
}
async function getimageB() {
  // give_image();
  const res = await fetch_imageB();
  const data = res.data;
  imageBattr.value = data;
  console.log(imageBattr.value);
}
onMounted(getimageB);
onMounted(fontlist);
onMounted(getimages);
async function getimages() {
  imageUrl1.value = await fetch_api("Result.png");
  imageUrl2.value = await fetch_api("ImageA.png");
}
const handleUpdate = (value: Array<GLfloat>) => {
  const res = change_attr(imageBattr.value, value);
  console.log(res);
  run_test();
  getimages();
};
</script>
