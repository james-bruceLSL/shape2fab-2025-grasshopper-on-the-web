<script setup>
import { ref } from "vue"
import GeometryView from "../components/MinimalisticGeometryView.vue"
import Slider from '../components/Slider.vue'
import Dropdown from "../components/Dropdown.vue"
import MetadataTextBox from "@/components/MetadataTextBox.vue"

//define path to grasshopper script
import def from "../assets/wall.gh"
const path = def

// declare inputs
const widthSliderName = ref("Width") 
const widthSliderValue = ref(10)

const heightSliderName = ref("Height")
const heightSliderValue = ref(14)

const VerticalRotationSliderName = ref("VerticalRotation") 
const VerticalRotationSliderValue = ref(10)

const HorizontalRotationSliderName = ref("HorizontalRotation")
const HorizontalRotationSliderValue = ref(10)

const ShapeDropDownName = ref("Shape")
const ShapeIndex = ref(0)
const dropdownOptions = ref([
  { label: "Standard", value: 0},
  { label: "Convex", value: 1},
  { label: "Concave", value: 2}
])


let metadata = ref([])

//Put all inputs together to send them to Rhino.Compute
let inputs = ref({
  [heightSliderName.value]: heightSliderValue.value,
  [widthSliderName.value]: widthSliderValue.value,
  [VerticalRotationSliderName.value]: VerticalRotationSliderValue.value,
  [HorizontalRotationSliderName.value]: HorizontalRotationSliderValue.value,
  [ShapeDropDownName.value]: ShapeIndex.value
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

<template>

  <div id="sidebar">
    <Slider :title = heightSliderName
    min = "1"
    max = "20"
    step = "1"
    :val = heightSliderValue
    @update = "updateValue"
    ></Slider>

    <Slider :title = widthSliderName
    min = "1"
    max = "20"
    step = "1"
    :val = widthSliderValue
    @update = "updateValue"
    ></Slider>

    <Slider :title = VerticalRotationSliderName
    min = "1"
    max = "20"
    step = "1"
    :val = VerticalRotationSliderValue
    @update = "updateValue"
    ></Slider>

    <Slider :title = HorizontalRotationSliderName
    min = "1"
    max = "20"
    step = "1"
    :val = HorizontalRotationSliderValue
    @update = "updateValue"
    ></Slider>

    <Dropdown :title="ShapeDropDownName"
    :options="dropdownOptions" :val="ShapeIndex" @update="updateValue"></Dropdown>
    <MetadataTextBox :metadata="metadata"></MetadataTextBox>
  
  </div>

  <div id="viewer">
    <!-- <GeometryView :data="inputs" :path="path" ></GeometryView> -->
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

</style>