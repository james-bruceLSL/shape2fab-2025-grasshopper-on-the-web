<template>
  <div id="body">
    <!-- HTML -->
    <h1 id="title">Introduction to Web Developments</h1>
    <br />
    <p>Web development is made up of three major parts:</p>

    <ul>
      <li>
        HTML: Outlines the content of a web page and defines its structure and
        hierarchy.
      </li>
      <li>
        CSS: Defines the style of html elements (ex. color, fonts, spacings)
      </li>
      <li id = "change-this">JavaScript: Adds interactivity and dynamic behavior.</li>
    </ul>

    <br />

    <div class="myButton" @click="changeTitleColor()">Click Me!</div>
    <br />

    <!-- <Slider
      title="Width of something"
      min="0"
      max="100"
      step="1"
      val="0"
    ></Slider> -->

    <br />

    <h2>
      To learn more about basic web development feel free to check out the
      following link:
    </h2>
    <a href="https://www.w3schools.com/html/html_intro.asp">W3 Link</a>

    <br />
    <br />

  </div>
</template>

<script setup>
import Slider from "../components/Slider.vue";
//Javascript

// Generate random hex color
const getRandomColor = () => {
  const letters = "0123456789ABCDEF";
  let color = "#";
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
};

const changeTitleColor = () => {
  const titleElement = document.getElementById("title");
  let newColor = getRandomColor();
  titleElement.style.color = newColor;

  //Add console.logs to see what's happening
  console.log("Change color", newColor);
};
</script>

<style>
/* CSS */
#body {
  display: flex;
  flex-direction: column;
  margin: 20px;
}

li {
font-family: 'Brandon Grotesque Light', serif;
  font-size: 20px;
  color: #333;
}

.myButton {
  background-color: black;
  color: white;
  width: 100px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
}
.myButton:hover {
  cursor: pointer;
}

#change-this {
  color: red;
  font-weight: bold;
  font-size: 20px;
  text-decoration: underline;
}
</style>
