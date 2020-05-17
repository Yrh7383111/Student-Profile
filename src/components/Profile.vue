<!-- Front-end -->


<!-- HTML -->
<template>
  <!-- With CSS below -->
  <div class="content">
    <!-- With CSS below -->
    <div class="collapse">
      <h1 style="padding: 25px 0 0 0">Student Profiles</h1>

      <!-- 1. Filter students by their names (both first and last name) -->
      <el-input
        id="name-input"
        placeholder="Search by name"
        prefix-icon="el-icon-search"
        v-model="inputName">
      </el-input>

      <!-- 2. Filter students by tag -->
      <el-input
        id="tag-input"
        placeholder="Search by tag"
        prefix-icon="el-icon-search"
        v-model="inputTag">
      </el-input>


      <!-- 3. Active student profiles -->
      <!-- With CSS below -->
      <el-collapse class="el-collapse">
        <!-- List rendering with single argument - iterate through "students" array -->
        <el-collapse-item v-for="item in filteredStudents" :key="item.id" :name="item.id">
          <!-- (1). Explicit student information without expanding the list -->
          <template slot="title">
            <!-- Avatar -->
            <div class="avatar">
              <div class="block"><el-avatar :size="100" :src="item.pic"></el-avatar></div>
            </div>
            <!-- Remaining information of a student -->
            <!-- With CSS below -->
            <div class="info">
              <h1>{{item.firstName.toUpperCase()}} {{item.lastName.toUpperCase()}}</h1>
              <p>Email: {{item.email}}</p>
              <p>Company: {{item.company}}</p>
              <p>Skill: {{item.skill}}</p>
              <p>Average: {{item.average}}%</p>
            </div>
          </template>

          <!-- With CSS below -->
          <!-- (2). Expand list -->
          <div class="expand-btn">
            <!-- List rendering with two arguments - iterate through "grades array and return index -->
            <!-- index - index of each test grade -->
            <p v-for="(g,index) in item.grades" :key="index">Test{{index}}:<i style="margin-left: 20px"></i>{{g}}%</p>
            <!-- List rendering with single argument - iterate all tags -->
            <el-tag
              :key="item.id + tag"
              v-for="tag in item.tags"
              :disable-transitions="false">
              {{tag}}
            </el-tag>

            <!-- (3). Create new tags -->
            <el-input
              class="input-new-tag add-tag-input"
              v-model="tagValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleNewTag(item.id)"
              @blur="handleNewTag">
            </el-input>
          </div>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>



<!-- CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .content
  {
    padding: 50px 250px;
    border-radius: 50px;
  }

  input.el-input__inner
  {
    width: 99%;
    border: unset;
    border-bottom: solid 1px;
    border-radius: unset;
  }

  .collapse
  {
    width: 100%;
    height: 500px;
    overflow-y: scroll;
    border-radius: 25px;
    background-color: white;
  }

  .el-collapse
  {
    padding: 20px;
    border:unset!important;
  }

  .el-collapse-item__header
  {
    height: auto!important;
    line-height: unset!important;
  }

  .info
  {
    text-align: left;
    margin-left: 50px;
  }

  .expand-btn
  {
    text-align: left;
    padding: 0 150px;
  }
</style>





<!-- Back-end -->
<script>
export default {
  name: 'Profile',

  // 1. Data model - OOP
  data() {
      return {
        allStudents: [],                                          // Copy of "students" array
        inputName: '',                                            // Input (string) to name search bar
        inputTag: '',                                             // Input (string) to tag search bar
        tagValue: ''}                                             // New tag value
  },

  // 2. Initialization of the program
  mounted() {
    // Fetch the data from public JSON API using "get" method
    this.$http.get('https://www.hatchways.io/api/assessment/students').then((res) => {
      // Students - an array of objects
      this.allStudents = res.data.students;
      // Map original "allStudents" array to a new array with two more variables
      // 1. tags: []
      // 2. average: ''
      this.allStudents.map(x => {
        x.tags = []; x.average = this.getAverage(x.grades)})
    }).catch((err) => {
      console.log(err)
    })
  },

  // 3. Computed property
  computed: {
    // New variable - filteredStudents
    filteredStudents: function()
    {
      // Regular expression from user input
      const inputName = new RegExp(this.inputName,'i');
      const inputTag = new RegExp(this.inputTag,'i');

      // Handle search by name using filter function
      const temp = this.allStudents.filter(item => {
        return (item.firstName + item.lastName).match(inputName)!= null});
      // Handle search by tag using filter function
      if (this.inputTag !== '')
      {
        return temp.filter((item) => {                                                      // Return an array of students with the same name and tag
          return item.tags.filter((tag) => {                                                // Return an array of students with the same tag
            return tag.match(inputTag) != null}).length!== 0})
      }
      else {
        return temp                                                                         // Only return an array of students with the same name
      }
    }
  },

  // 4. Helper functions
  methods: {
    // Calculate the average of grades
    getAverage(array)
    {
      let sum = 0;
      array.map(x => sum += Number(x));
      return (sum / array.length);
    },

    // Handle the new tag
    handleNewTag(index)
    {
      const i = Number(index) - 1;
      const tagValue = this.tagValue;

      if (tagValue)
      {
        this.filteredStudents[i].tags.push(tagValue);                                       // Push the tag to "tags" array
      }
      this.tagValue = '';                                                                   // Free up "tagValue"
    }
  }
}
</script>