<template>
  <div>
    <div class="search-input mx-3">
      <div class="input-group input-group-lg">
        <input
          type="text"
          class="form-control"
          v-model="students_filter"
          placeholder="Search by name"
        />
      </div>
      <div class="input-group input-group-lg mt-3">
        <input
          type="text"
          class="form-control"
          v-model="tags_filter"
          placeholder="Search by tags"
        />
      </div>
    </div>
    <div
      v-for="student in filterStudentsByName"
      :key="student.id"
      class="student-card mt-3"
    >
      <b-row>
        <b-col>
          <b-avatar
            :src="student.pic"
            :alt="student.firstName"
            class="circular-image rounded-circle mt-3"
            size="8rem"
          >
          </b-avatar>
        </b-col>
        <b-col cols="9">
          <div>
            <div class="d-flex justify-content-between">
              <h5 class="student-name">
                {{ student.firstName }} {{ student.lastName }}
              </h5>
              <div
                v-on:click="toggleGrades(student.id)"
                class="toggle-grades mr-3"
                :id="'toggle_grades_' + student.id"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="30"
                  height="30"
                  viewBox="0 0 30 30"
                  fill="#a6a6a6"
                  :id="'plus_sign_' + student.id"
                >
                  <path d="M24 9h-9v-9h-6v9h-9v6h9v9h6v-9h9z" />
                </svg>
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="30"
                  height="30"
                  viewBox="0 0 30 30"
                  fill="#a6a6a6"
                  :id="'minus_sign_' + student.id"
                  class="d-none"
                >
                  <path d="M0 9h24v6h-24z" />
                </svg>
              </div>
            </div>
            <div class="details-section ml-4 mt-2">
              <div class="student-details">
                <p class="student-email">
                  Email: <span>{{ student.email }}</span>
                </p>
                <p class="student-company">
                  Company: <span>{{ student.company }}</span>
                </p>
                <p>
                  Skill: <span>{{ student.skill }}</span>
                </p>
                <p v-cloak>
                  Average: <span>{{ getAverage(student.grades) }}%</span>
                </p>
                <div
                  class="grades-section mt-4"
                  :id="'grades_section_' + student.id"
                >
                  <p v-for="(grade, index) in student.grades" :key="index">
                    Test-{{ index + 1 }}: {{ grade }}%
                  </p>
                  <div class="d-flex flex-nowrap mb-3" v-if="tags[student.id]">
                    <div
                      v-for="(tag, index) in tags[student.id]"
                      :key="index"
                      class="student-tag"
                    >
                      {{ tag }}
                    </div>
                  </div>
                  <input
                    type="text"
                    placeholder="Add a tag"
                    class="tag-input mt-2"
                    @keydown.enter="addTag(student.id, $event)"
                  />
                </div>
              </div>
            </div>
          </div>
        </b-col>
      </b-row>
      <hr />
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import $ from "jquery";
export default {
  name: "UserDetail",
  data() {
    return {
      students: [],
      tags: {},
      students_filter: "",
      tags_filter: "",
    };
  },
  methods: {
    getStudents() {
      let endpoint = "https://www.hatchways.io/api/assessment/students";
      this.loadingStudents = true;
      fetch(endpoint).then(async (response) => {
        const data = await response.json();
        this.students = data.students;
      });
    },
    getAverage(grades) {
      var n = grades.length;
      var count = 0;
      for (var i = 0; i < n; i++) {
        count += parseInt(grades[i]);
      }
      return count / n;
    },
    toggleGrades(id) {
      var x = document.querySelector("#grades_section_" + id);
      if (x.style.display === "block") {
        x.style.display = "none";
        $("#plus_sign_" + id).removeClass("d-none");
        $("#minus_sign_" + id).addClass("d-none");
      } else {
        x.style.display = "block";
        $("#plus_sign_" + id).addClass("d-none");
        $("#minus_sign_" + id).removeClass("d-none");
      }
    },
    addTag(id, event) {
      event.preventDefault();
      var val = event.target.value.trim();

      if (val.length > 0) {
        if (id in this.tags) {
          var old_tags = this.tags[id];
          old_tags.push(val);
          Vue.set(this.tags, id, old_tags);
          // this.tags[id].push(val);
        } else {
          Vue.set(this.tags, id, [val]);
        }
        event.target.value = "";
      }
    },
  },
  created() {
    this.getStudents();
    document.title = "students list";
  },
  computed: {
    filterStudentsByName() {
      return this.students.filter((student) => {
        const first_name = student.firstName.toString().toLowerCase();
        const last_name = student.lastName.toString().toLowerCase();
        let student_tags = "";
        if (student.id in this.tags) {
          student_tags = this.tags[student.id].toString().toLowerCase();
        }

        const nameSearchTerm = this.students_filter.toLowerCase();
        const tagSearchTerm = this.tags_filter.toLowerCase();
        if (nameSearchTerm) {
          return (
            first_name.includes(nameSearchTerm) ||
            last_name.includes(nameSearchTerm)
          );
        }
        if (tagSearchTerm) {
          return student_tags.includes(tagSearchTerm);
        } else {
          return (
            first_name.includes(nameSearchTerm) ||
            last_name.includes(nameSearchTerm)
          );
        }
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* image */
.circular-image {
  border: 1px solid lightgray;
  display: block;
  margin-left: auto;
  margin-right: auto;
}
.badge-secondary {
  background-color: white;
}

.student-name {
  text-transform: uppercase;
  font-size: 2.2em;
  font-family: Arial, Helvetica, sans-serif;
  font-weight: bold;
}

.student-details {
  font-family: sans-serif;
  font-weight: 400;
  line-height: 0.6rem;
  color: gray;
}

.grades-section {
  display: none;
  line-height: 0.4rem;
}

.student-tag {
  background-color: lightgray;
  padding: 12px;
  margin-right: 5px;
  border-radius: 3px;
}

input {
  border: 0;
  border-bottom: 2px solid lightgray;
  font-family: sans-serif;
  font-weight: 400;
}

.input-group input {
  border: 0;
  border-bottom: 1px solid lightgray;
  border-radius: unset;
  padding-inline: unset !important;
  padding-bottom: unset !important;
}

input:hover,
input:active,
input:focus {
  outline: 0px !important;
  -webkit-appearance: none;
  box-shadow: none !important;
}

input:focus {
  border: 0;
  border-bottom: 2px solid black;
}
</style>
