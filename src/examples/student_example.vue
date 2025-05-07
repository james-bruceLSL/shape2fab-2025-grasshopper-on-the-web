
<script setup>

import { ref } from "vue";

import GeometryView from "../components/BurgerGeometryViewFull.vue"
import Toggle from "../components/Toggle.vue"
import Slider from "../components/Slider.vue"
// import Dropdown from "../components/Dropdown.vue"
import MetadataTextBox from "@/components/MetadataTextBox.vue"

import def from '../assets/RC_floor_options.gh' 
const path = def //path to the Grasshopper definition

// declare inputs
const XSliderName = ref("XGrid") 
const XSliderValue = ref(7) //default value in metres

const YSliderName = ref("YGrid") 
const YSliderValue = ref(5) //default value in metres 

const SquareToggleName = ref("SquareBay") 
const SquareToggleValue = ref(false)

// const dropdownName = ref("Dropdown")
// const dropdownIndex = ref(0)
// const dropdownOptions = ref([
//   {label: "Option 1", value: 0},
//   {label: "Option 2", value: 1},
//   {label: "Option 3", value: 2}
// ])

let metadata = ref([])

//declare the inputs to send to compute here
let inputs = ref({
  [XSliderName.value]: XSliderValue.value,
  [YSliderName.value]: YSliderValue.value,
  [SquareToggleName.value]: SquareToggleValue.value
});

function updateValue(newValue, parameterName) {
  // Iterate over the inputs array
  for (const [key, value] of Object.entries(inputs.value)) {
    if (key == parameterName){
      inputs.value[key] = newValue
    }
  }
}

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}


</script>

<!-- Template is a HTML-based syntax that allows you to bind the rendered DOM elements
with data, objects, functions etc. -->
<template>

    
  <div id="sidebar" >

      <h2> RC Scheme designer </h2>
      <hr>

      <p>See impact of column spacing on embodied carbon for RC schemes </p>
      <br>

      <h2> Inputs</h2>
      <hr>

      <Slider
      :title="XSliderName"
      min="4"
      max="12"
      step="0.1"
      :val="XSliderValue"
      @update="updateValue"
      ></Slider>

      <Slider
      :title="YSliderName"
      min="4"
      max="12"
      step="0.1"
      :val="YSliderValue"
      @update="updateValue"
      ></Slider>

      <Toggle
      :title="SquareToggleName"
      :val="SquareToggleValue"
      @update= "updateValue"
      ></Toggle>


      <hr>

      <MetadataTextBox :metadata="metadata"></MetadataTextBox>

  </div>

  <div id="viewer">
    <GeometryView 
    :data="inputs" 
    :path="path" 
    @updateMetadata="receiveMetadata"></GeometryView>
  </div>

</template>


<style scoped>

#sidebar {
  width: 350px;
  padding: 20px;
  flex-shrink: 0; 
}
 
#viewer { 
  width: 500px
} 

html[data-theme='dark'] .sidebar {
  background-color: rgba(0, 0, 0, 0.7) !important;
}

</style>