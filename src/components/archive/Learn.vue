<template>

  <div class="background">
      <validate v-if="personValidate.selected"></validate>
      <info v-if="personInfo.selected"></info>
      <project v-if="projectManage.selected"></project>
    <el-row class="bottom-tab">
      <learntab v-on:changetab="selectTab(3)" :info="projectManage"></learntab>
      <learntab v-on:changetab="selectTab(1)" :info="personValidate"></learntab>
      <learntab v-on:changetab="selectTab(2)" :info="personInfo" ></learntab>

    </el-row>
  </div>
</template>

<script>
import learntab from './LearnTab'
import validate from './PersonValidate'
import info from './PersonInfo'
import project from './PersonProject'
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      personValidate: {
        tabContent: '身份认证',
        imgUrl: '/static/validate.svg',
        selected: true
      },
      personInfo: {
        tabContent: '个人信息',
        imgUrl: '/static/person.svg',
        selected: false
      },
      projectManage: {
        tabContent: '项目管理',
        imgUrl: '/static/project.svg',
        selected: false
      },
      currentSelected: 1
    }
  },
  components: {
    learntab,
    validate,
    info,
    project
  },
  methods: {
    selectTab: function (tabIndex) {
      if (tabIndex === 1) {
        this.personValidate.selected = true
      } else {
        this.personValidate.selected = false
      }
      if (tabIndex === 2) {
        this.personInfo.selected = true
      } else {
        this.personInfo.selected = false
      }
      if (tabIndex === 3) {
        this.projectManage.selected = true
      } else {
        this.projectManage.selected = false
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.bottom-tab{
  font-size: 14px;
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 8%;
  /*background-color: rgb(242,242,242);*/
  background-color: #eee;
  /*border-top: 1px solid rgb(153,153,153);*/
}
.background{
  width: 100%;
  height: 100%;
  background-color: #fff;
  /*border-radius: 7px;
  border: 2px solid #fff;*/
  position: absolute;
  left: 0;
}

</style>
