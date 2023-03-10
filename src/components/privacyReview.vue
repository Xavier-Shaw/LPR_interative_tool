<template>
  <div id="formDiv" style="width: 100%">
    <h1>Lean Privacy Review Form</h1>
    <el-row justify="center">
      <el-card style="width: 30%; margin: 20px">
        <el-row justify="center">
          <el-col :span="12">
            <h4 style="margin-top: 5px">User Name: </h4>
          </el-col>
          <el-col :span="12">
            <el-input label="User Name" v-model="userName" placeholder="Please input your name"/>
          </el-col>
        </el-row>
      </el-card>
    </el-row>


    <h3>Rate Your Privacy Concern From 1 to 5 and Write Down Your Comment</h3>
    <p>(1 indicates the most comfortable & 5 indicates the most uncomfortable)</p>
    <div style="margin: 20px;" v-for="concernInfo in concernData" v-bind:key="concernInfo">
      <el-card shadow="always">
        <el-row :gutter="10">
          <el-col :span="6">
            <h4 style="margin-top: 5px; ">{{concernInfo.content}}</h4>
          </el-col>
          <el-col :span="6">
            <el-radio-group v-model="concernInfo.level">
              <el-radio-button label="0">NaN</el-radio-button>
              <el-radio-button label="1"></el-radio-button>
              <el-radio-button label="2"></el-radio-button>
              <el-radio-button label="3"></el-radio-button>
              <el-radio-button label="4"></el-radio-button>
              <el-radio-button label="5"></el-radio-button>
            </el-radio-group>
          </el-col>
          <el-col :span="12">
            <el-input
                v-model="concernInfo.comment"
                autosize
                type="textarea"
                placeholder="Please input your comment"
            />
          </el-col>
        </el-row>
      </el-card>
    </div>

    <el-button type="primary" @click="submitForm">Submit the Form</el-button>
    <el-button type="primary" @click="generateGridChart">Generate Grid Chart</el-button>
    <el-button type="primary" @click="reset">Reset</el-button>
    <div style="margin-top: 20px">
      <span>Group Size: </span>
      <el-input-number style="width: 100px; margin-right: 30px" :precision="0" :min="1" :max="10" v-model="groupSize"></el-input-number>
      <p>(Please press "Reset" after you change the group size)</p>
      <h4>Data below is from user #{{startUserId + 1}} to user #{{endUserId}} ({{userConcernsData.length}} users in total)</h4>
      <el-button type="primary" @click="showNextGroup">Show Next Group of Users</el-button>
    </div>

  </div>

  <div id="gridDiv" style="width: 90%; margin-top: 50px">
    <h2 style="margin-left: 150px">Privacy Concerns From Different Users</h2>

    <svg id="colorGradient"></svg>

    <br>

    <svg id="gridChart">

    </svg>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "privacyReview",

  data() {
    let concernData = [];
    const privacyConcerns = ["Invasive monitoring", "Violation of expectations", "Lack of respect for autonomy", "Lack of informed consent", "Deceptive data practice", "Lack of protection for vulnerable population", "Lack of alternative choice", "Insufficient data security", "Insufficient anonymization", "Too high potential risks", "Bias or discrimination", "Lack of trust for algorithms", "Lack of control of personal data", "Data commodification", "Unclassified", "No privacy concern"]
    let concernLevel = 0;
    let concernComment = "";

    for (const concern of privacyConcerns) {
      concernData.push({content: concern, level: concernLevel, comment: concernComment});
    }

    return {
      privacyConcerns,
      userName: "",
      concernData,
      userId: 0,
      userConcernsData: [],
      userNames: [],
      gridSize: 35,
      colorArray: ['#FFFFFF' , '#aac4de', '#99bd96', '#d2cdac', '#85634e', '#2c2019'],
      color: '#2c2019',
      startUserId: 0,
      endUserId: 0,
      groupSize: 10
    }
  },

  methods: {
    submitForm() {
      this.userId += 1
      let storeData = JSON.parse(JSON.stringify(this.concernData))
      this.userConcernsData.push({userName: this.userName, userId: this.userId, concernData: storeData})

      //reset form
      this.userName = ""
      for (const data of this.concernData) {
        data.level = 0
        data.comment = ""
      }

      this.saveDataToFile()
    },

    generateColorGradient() {
      let colors = ['#aac4de', '#99bd96', '#d2cdac', '#85634e', '#2c2019']

      let svg = d3.select('#colorGradient')
          .style("margin-left", 180)
          .attr('width', 650)
          .attr('height', 60);

      let grad = svg.append('defs')
          .append('linearGradient')
          .attr('id', 'grad')
          .attr('x1', '0%')
          .attr('x2', '100%')
          .attr('y1', '0%')
          .attr('y2', '0%');

      grad.selectAll('stop')
          .data(colors)
          .enter()
          .append('stop')
          .style('stop-color', function(d){ return d; })
          .attr('offset', function(d,i){
            return 100 * (i / (colors.length - 1)) + '%';
          })

      svg.append('rect')
          .attr('x', 60)
          .attr('y', 20)
          .attr('width', 550)
          .attr('height', 25)
          .style('fill', 'url(#grad)');

      svg.append('text')
          .attr('x', 0)
          .attr('y', 15)
          .text('Most Comfortable')

      svg.append('text')
          .attr('x', 500)
          .attr('y', 15)
          .text('Most Uncomfortable')
    },

    clearGridChart() {
      d3.select("#gridChart").selectAll('*').remove()
    },

    reset() {
      this.clearGridChart()
      this.startUserId = 0
      this.endUserId = Math.min(this.groupSize, this.userConcernsData.length)
    },

    showNextGroup() {
      this.clearGridChart()
      if (this.endUserId !== this.userConcernsData.length) {
        this.startUserId = this.endUserId
        this.endUserId = Math.min(this.startUserId + this.groupSize, this.userConcernsData.length)
      }
      this.generateGridChart()
    },

    generateGridChart() {
      let textWidth = 250
      let padding = 30
      let width = this.gridSize * 20 + textWidth
      let height = this.gridSize * (this.privacyConcerns.length + 3) + padding

      let svg = d3.select("#gridChart")
          .attr("width", width)
          .attr("height", height)

      // Define the data for the grid
      const data = this.userConcernsData.slice(this.startUserId, this.endUserId);
      const data_names = this.userNames.slice(this.startUserId, this.endUserId);

      const xScale = d3.scaleBand()
          .domain(data_names)
          .range([textWidth, width])
          .padding(0.1);

      const yScale = d3.scaleBand()
          .domain(this.privacyConcerns)
          .range([0, height - padding])
          .padding(0.1)

      const xAxisTransform = `translate(0, ${height - padding})`
      const yAxisTransform = `translate(${textWidth}, 0)`
      // const gridTransform = `translate(${textWidth}, ${height - padding}`

      const xAxis = d3.axisBottom(xScale)
      svg.append("g")
          .attr("transform", xAxisTransform)
          .call(xAxis);

      const yAxis = d3.axisLeft(yScale)
      svg.append("g")
          .attr("transform", yAxisTransform)
          .call(yAxis)



      for (const userData of data) {
        // const userId = userData.userId
        const concernData = userData.concernData

        const gridGroup = svg.append("g").attr("id", userData.userName);

        gridGroup.selectAll("g")
            .data(concernData)
            .enter().append("rect")
            .attr("class", "grid")
            .attr("width", xScale.bandwidth)
            .attr("height", yScale.bandwidth)
            .attr("x", xScale(userData.userName))
            .attr("y", (d) => yScale(d.content))
            // .attr("transform", gridTransform)
            .attr('fill', (d) => {
              return this.colorArray[d.level % 6]
            })
            .append('title')
            .text((d) => d.comment)
      }

    },

    saveDataToFile() {
      // Convert the data object to a JSON string
      const json = JSON.stringify(this.userConcernsData);

      // Create a new Blob object with the JSON data
      const blob = new Blob([json], {type: "application/json"});

      // Create a new anchor element to download the file
      const anchor = document.createElement("a");
      anchor.download = "data.json";
      anchor.href = URL.createObjectURL(blob);
      anchor.dataset.downloadurl = [
        "application/json",
        anchor.download,
        anchor.href,
      ].join(":");

      // Trigger a click event on the anchor element to download the file
      anchor.click();

      // Revoke the object URL to free up memory
      URL.revokeObjectURL(anchor.href);
    },

    async loadDataFromFile() {
      const response = await fetch("/data.json");
      const text = await response.text();
      this.userConcernsData = await JSON.parse(text);
      for (const data of this.userConcernsData) {
        this.userNames.push(data.userName)
      }
      this.userId = this.userConcernsData.length
      this.startUserId = 0
      this.endUserId = Math.min(this.groupSize, this.userConcernsData.length)
      console.log(this.userConcernsData)
    },
  },

  mounted() {
    this.loadDataFromFile()
    this.generateColorGradient()
  }
}
</script>

<style scoped>

</style>