<template>
  <div class="wrapper" style="max-width: 90%">
    <el-form label-width="60px">
      <el-form-item>
        <h1 class="title">База заданий</h1>
        <h3>Таблица курсов</h3>
        <el-table
          :data="
            courses.filter(
              (data) =>
                !search ||
                data.name.toLowerCase().includes(search.toLowerCase())
            )
          "
          :default-sort="{ prop: 'creation_time', order: 'descending' }"
          style="width: 100%"
        >
          <el-table-column label="Название курса" prop="name">
          </el-table-column>
          <el-table-column label="Число студентов" prop="students_count">
          </el-table-column>
          <el-table-column label="Число преподавателей" prop="teachers_count">
          </el-table-column>
          <el-table-column
            label="Дата создания курса"
            prop="human_creation_dt"
            sortable
          >
          </el-table-column>
          <el-table-column label="Действия">
            <template slot-scope="scope">
              <el-button
                size="mini"
                @click="openCourse(scope.$index, scope.row)"
                >Перейти к курсу</el-button
              >
            </template>
          </el-table-column>
          <el-table-column align="right">
            <template slot="header" slot-scope="scope">
              <el-input
                v-model="search"
                size="mini"
                placeholder="Введите название курса для поиска"
              />
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>

      <el-form-item>
        <h3>Таблица КИМов</h3>
        <el-table
          :data="
            cmms.filter(
              (data) =>
                !searchCMM ||
                data.name.toLowerCase().includes(searchCMM.toLowerCase())
            )
          "
          :default-sort="{ prop: 'created_at', order: 'descending' }"
          style="width: 100%"
          max-height="300"
        >
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-button
                @click.native.prevent="openCMM(scope.row.spreadsheet_link)"
                size="small"
              >
                Открыть КИМ
              </el-button>
              <el-button
                @click.native.prevent="manageCMM(scope.row.id, scope.row.name)"
                size="small"
              >
                Управление КИМом
              </el-button>
              <el-button
                @click.native.prevent="deleteCMM(scope.row.id, cmms)"
                type="danger"
                size="small"
              >
                Удалить КИМ
              </el-button>
            </template>
          </el-table-column>
          <el-table-column prop="name" label="Название КИМа"> </el-table-column>
          <el-table-column prop="description" label="Описание КИМа">
          </el-table-column>
          <el-table-column
            prop="human_creation_dt"
            label="Дата создания КИМа"
            sortable
          >
          </el-table-column>
          <el-table-column align="right">
            <template slot="header" slot-scope="scope">
              <el-input
                v-model="searchCMM"
                size="mini"
                placeholder="Введите название КИМа для поиска"
              />
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" :disabled="cmmLoading" @click="addCMM()"
          >Добавить КИМ</el-button
        >
        <p></p>
        <div v-if="clickedAddCMM">
          <el-input
            type="text"
            placeholder="Название КИМа"
            v-model="cmmName"
            maxlength="100"
            show-word-limit
          >
          </el-input>
          <div style="margin: 20px 0"></div>
          <el-input
            type="textarea"
            placeholder="Описание КИМа"
            v-model="cmmDescription"
            maxlength="300"
            show-word-limit
          >
          </el-input>
          <p></p>
          <el-button :loading="cmmLoading" @click="createCMM()"
            >Создать КИМ</el-button
          >
        </div>
      </el-form-item>
    </el-form>

    <!--h3>Таблица грейдеров</h3>
    <el-table :data="courses" style="width: 100%" max-height="300">
      <el-table-column fixed prop="name" label="Название грейдера" width="250">
      </el-table-column>
      <el-table-column prop="alternate_link" label="Ссылка на грейдер" width="200">
      </el-table-column>
      <el-table-column fixed="right" label="Operations" width="300">
        <template slot-scope="scope">
          <el-button
            @click.native.prevent="openCMM(scope.$index, cmms)"
            type="text"
            size="small"
          >
            Опубликовать грейдер
          </el-button>
        </template>
      </el-table-column>
    </el-table-->
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      cmms: [],
      courses: [],
      tasks: [],
      currentCmm: "",
      search: "",
      searchCMM: "",
      cmmName: "",
      cmmDescription: "",
      cmmLoading: false,
      clickedAddCMM: false,
    };
  },
  beforeMount() {
    this.getCourses();
    this.getCMMs();
    this.getTasks();
  },
  methods: {
    getCMMs() {
      const path = "https://constructor.auditory.ru/cmms";
      axios.get(path, { withCredentials: true }).then(
        (res) => {
          this.cmms = res.data.cmms;
          this.cmms.forEach(function (element) {
            var temp = new Date(element.created_at);
            element.human_creation_dt = temp.toLocaleString();
          });
        },
        (error) => {
          console.error(error);
        }
      );
    },
    getCourses() {
      const path = "https://constructor.auditory.ru/courses";
      axios.get(path, { withCredentials: true }).then(
        (res) => {
          this.courses = res.data.courses;
          this.courses.forEach(function (element) {
            var temp = new Date(element.creation_time);
            element.human_creation_dt = temp.toLocaleString();
          });
        },
        (error) => {
          console.error(error);
        }
      );
    },
    getTasks() {
      const path = "https://constructor.auditory.ru/tasks";
      axios.get(path, { withCredentials: true }).then(
        (res) => {
          this.tasks = res.data.tasks;
        },
        (error) => {
          console.error(error);
        }
      );
    },
    addCMM() {
      (this.clickedAddCMM = true), this.createCMM;
    },
    createCMM() {
      if (this.cmmName.length >= 3) {
        this.cmmLoading = true;
        this.$message({
          showClose: true,
          message: "КИМ создаётся...",
        });
        axios
          .post(
            "https://constructor.auditory.ru/cmms",
            {
              name: this.cmmName,
              description: this.cmmDescription,
            }
          )
          .then(
            (res) => {
              console.log(res);
              if (res.data.status === "success") {
                this.cmms.push(res.data.cmm);
                this.getCMMs();
                this.$message({
                  showClose: true,
                  message: "КИМ успешно создан!",
                  type: "success",
                });
                this.clickedAddCMM = false;
                this.cmmLoading = false;
                this.cmmName = "";
                this.cmmDescription = "";
              } else {
                this.$message({
                  showClose: true,
                  message: "Ошибка при создании КИМа",
                  type: "error",
                });
                this.clickedAddCMM = false;
                this.cmmLoading = false;
              }
            },
            (error) => {
              console.error(error);
            }
          );
      } else {
        if (this.cmmName.length == 0)
          this.$message({
            showClose: true,
            message: "Введите название КИМа!",
            type: "error",
          });
        else
          this.$message({
            showClose: true,
            message: "Слишком короткое название для КИМа!",
            type: "error",
          });
      }
    },
    openCMM(link) {
      window.open(link, "_blank");
    },
    manageCMM(CMMid, CMMname) {
      this.$router.push({
        name: "Manage CMM",
        params: { id: CMMid, name: CMMname },
      });
    },
    openCourse(index) {
      window.open(this.courses[index].alternate_link, "_blank");
    },
    deleteCMM(id, cmms) {
      let i = cmms.map((item) => item.id).indexOf(id);
      this.$delete(this.cmms, i);
      const path = `https://constructor.auditory.ru/cmms/${id}`;
      axios
        .delete(path)
        .then(() => {
          this.getCMMs();
          this.$message({
            showClose: true,
            message: "КИМ успешно удалён!",
            type: "success",
          });
        })
        .catch((error) => {
          console.error(error);
          this.getCMMs();
          this.$message({
            showClose: true,
            message: "При удалении КИМа возникла ошибка!",
            type: "error",
          });
        });
    },
  },
};
</script>
