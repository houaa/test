<template>
  <div id="Container">
    <div style="justify-content: space-between;">
      <div id="Meta">
        <div id="Name">
          <div placeholder="修改姓名" style="outline:none;font-weight: 800;font-size: 24px;color: rgb(11, 178, 121);width: 95px;border: none;">{{user.name}}</div>
          <div v-if="user.sex" style="display: inline-block;position:relative;top:4px;margin-left: 5px;"><img width="20" src="../assets/male.svg"></div>
          <div v-else style="display: inline-block;position:relative;top:4px;margin-left: 5px;"><img width="20" src="../assets/female.svg"></div>
        </div>
        <div id="DetailMeta" style="margin-top: 20px;">
          <!-- <div id="auth" style="color: #0bb279;font-size: 14px;text-align:center">
                                      {{user.auth?"认证教员":"非认证教员"}}
                                    </div> -->
          <div id="grade" style="font-size: 14px;">
            <select v-on:change="preventWindow" v-model="user.edu"  id="eduSelector" style="outline:none;color: rgb(187, 187, 187);margin-right:5px;">
              <option v-for="i in Array.from(Array(4).keys())" v-bind:value="i" style="outline:none;">
                {{edu[i]}}
              </option>
            </select>
            <select v-on:change="preventWindow" v-model="user.grade"  id="gradeSelector" style="color: #777;outline:none;">
              <option v-for="i in Array.from(Array(grades[user.edu].length).keys())" v-bind:value="i" style="outline:none">
                {{grades[user.edu][i]}}
              </option>
            </select>
          </div>
          <div v-if="user.auth" id="auth">
            <img width="20" style="padding-top:3px;" src="../assets/auth.svg">
          </div>
          <div v-else id="auth" @click="handleAuthClick">
            未认证
          </div>
        </div>
      </div>
      <div id="rate" style="text-align: right;font-size: 23px;color: #0bb279; font-weight: 600;">
        <div style="color: #000;font-weight: 300;font-size:16px;padding-top: 5px;">
          注册“猴啊”<span style="font-weight: 400;color:rgb(11, 178, 121)">
            <!-- {{new Date()}}
            {{user.createdAt}} -->
            {{Math.floor(((new Date()) - user.createdAt)/3600000/24)}}天
            </span>
        </div>
        <div style="margin-top:15px;letter-spacing:2px;">
          <!-- {{user.rate}}
          <i style="font-size: 12px;font-style: normal; font-weight: 400;"> 分</i> -->
        </div>
      </div>
    </div>
    <div class="Content1">
      <div>薪资</div>
      <div style="font-size:16px">
        <i style="color: #0bb279;font-style: normal;">￥
          <input v-on:change="preventWindow" class="salaryInput" type="number" v-model.number="user.salary"></input>
        </i>/小时</div>
    </div>
    <div class="Content1">
      <div>校区</div>
      <div>
        <select id="campusSelector" v-model="user.campus"  v-on:change="preventWindow" >
          <option v-for="i in campus">
            {{i}}
          </option>
        </select>
      </div>
    </div>
    <div class="Content1" style="flex-direction: column;">
      <div>科目</div>
      <div id="class">
        <div class="eduRank" v-for="(subjects, i) in user.teach">
          <div>{{subjects.name}}</div>
          <div id="classes">
            <i v-for="(subject, j) in subjects.subjects" @click="toggleTeach([i, j])" v-bind:class="subject.checked?'ok':'not'">
              {{subject.name}}
            </i>
          </div>
        </div>
      </div>
    </div>
    <div class="week">
      <div class="time">
        <div>&nbsp;&nbsp;</div>
        <div>
          上午
        </div>
        <div>
          下午
        </div>
        <div>
          晚上
        </div>
      </div>
      <div v-for="i in [0,1,2,3,4,5,6]">
        <div>
          {{days[i]}}
        </div>
        <div v-for="j in [0,1,2]" @click="toggleCalendar([i,j])">
          <div class="time" v-bind:class="user.availableTime[i][j]?'okTime':'notTime'"><i v-if='user.availableTime[i][j]' style="color:#FFF;font-size: 10px;padding-top:7px;" class="el-icon-check"></i></div>
        </div>
      </div>
    </div>
    <div class="Content2">
      <div>
        标签
      </div>
      <div style="">
        <el-tag :key="tag" v-for="tag in user.tags" :closable="true" :close-transition="false" @close="deleteTag(tag)" style="text-overflow: ellipsis;max-width: 100px;overflow: hidden;font-weight:600;font-size:14px;display: inline-block;border-radius: 5px;background:#0bb279;margin-right: 9px;color:#FFF;margin-bottom: 10px;">
          {{tag}}
        </el-tag>
        <el-input v-on:change="preventWindow" class="input-new-tag" v-if="newTagInputVisible" v-model="newTag" ref="saveTagInput" size="mini" @keyup.enter.native="handleNewTagConfirm" @blur="handleNewTagConfirm">
        </el-input>
        <el-button v-else class="button-new-tag" style="font-weight:600;font-size:14px;" size="small" @click="showNewTagInput">+ New Tag</el-button>
      </div>
    </div>
    <div class="Content2">
      <div>
        个人宣言
      </div>
      <el-input v-on:change="preventWindow" placeholder="输入个人宣言" autosize type="textarea" v-model="user.selfIntro" id="SelfIntro" style="font-size:14px;color:#7e7e7e">
      </el-input>
    </div>
    <div style="width: 100%;">
      <el-button style="margin-left:auto;margin-right:auto;background:rgb(11, 178, 121)" type="success" @click="submit">提交</el-button>
    </div>
  </div>
</template>
<script>
// import AV from 'leancloud-storage'
import { mapGetters, mapActions, mapMutations } from 'vuex'

window.addEventListener('beforeunload', function (e) {
  if (window.preventWindowClose) {
    e.returnValue = 'Changes you made may not be saved.'
    return 'Changes you made may not be saved.'
  }
})

export default {
  'name': 'Self',
  data() {
    return {
      campus: [ '紫金港校区', '玉泉校区', '西溪校区', '华家池校区', '之江校区', '舟山校区' ],
      edu: ['初中', '高中', '小学', '本科'],
      grades: [['初一', '初二', '初三'], ['高一', '高二', '高三'], ['一年级', '二年级', '三年级', '四年级', '五年级', '六年级'], ['大一', '大二', '大三', '大四']],
      classes: [['全科', '陪读'], ['数学', '科学', '英语'], ['数学', '理综', '英语']],
      days: ['一', '二', '三', '四', '五', '六', '日'],
      newTag: '',
      newTagInputVisible: false,
      teacherOrStudent: true
    }
  },
  computed: {
    ...mapGetters([
      'user'
    ])
  },
  created: function () {
    fetch('https://api.houaa.xyz/person/me/', {
      method: 'GET',
      credentials: 'include'
    }).then(raw => raw.json())
    .then(json => {
      if (json.status === 'error') {
        this.$message(json.payload)
        this.$router.push('/login')
      } else {
        this.getInfo(json.payload)
      }
    }).catch(err => {
      console.log(err)
      this.$router.push('/login')
    })
  },
  methods: {
    ...mapActions([
      'getInfo',
      'submitToBackend'
    ]),
    ...mapMutations([
      'toggleTeach',
      'toggleCalendar',
      'deleteTag',
      'addTag',
      'inputText'
    ]),
    handleAuthClick() {
      this.$alert('认证功能即将上线')
    },
    showNewTagInput() {
      this.newTagInputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleNewTagConfirm() {
      window.preventWindowClose = true
      if (this.newTag) {
        this.addTag(this.newTag)
      }
      this.newTag = ''
      this.newTagInputVisible = false
    },
    preventWindow() {
      window.preventWindowClose = true
    },
    async submit() {
      if (this.user.salary <= 300 && this.user.salary >= 60) {
        this.inputText(['salary', this.user.salary])
        this.inputText(['selfIntro', this.user.selfIntro])
        console.log(this.user.grade)
        this.inputText(['grade', this.user.grade])
        this.inputText(['edu', this.user.edu])
        window.preventWindowClose = false
        try {
          await this.submitToBackend()
          this.$message('修改已提交')
        } catch (err) {
          console.log(err)
          this.$message(err.message)
        }
      } else {
        this.$message('薪水应当大于60并小于300')
      }
    }
  }
}
</script>
<style scoped>
#Container {
  width: 80%;
  margin-left: auto;
  margin-right: auto;
  color: #7e7e7e;
}

#Container>div {
  display: flex;
  border-bottom: 1px solid rgb(238, 238, 238);
  padding-bottom: 20px;
  padding-top: 20px;
}

#DetailMeta {
  display: flex;
  flex-direction: row;
}

.Content1 {
  display: flex;
  justify-content: space-between;
}

.Content1>:first-child {
  font-size: 1.2em;
  font-weight: 600;
  color: #000;
}

.button-new-tag {
  margin-left: 10px;
  height: 24px;
  line-height: 22px;
  padding-top: 0;
  padding-bottom: 0;
}

.input-new-tag {
  width: 78px;
  margin-left: 10px;
  height: 24px;
}

.Content1 .salaryInput {
  border: none;
  outline: none;
  width: 30px;
  text-align: left;
  font-size: 17px;
  margin-right: 5px;
  color: #0bb279;
  font-weight: 600;
}

#Container>div.Content2 {
  display: block;
}

.Content2> :first-child {
  display: block;
  cursor: pointer;
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 20px;
  color: #000;
}

.eduRank {
  display: block;
  margin-top: 20px;
}

.eduRank> :first-child {
  font-size: 15px;
  line-height: 30px;
  margin-left: 20px;
  display: inline-block;
  vertical-align: top;
}

.eduRank> :last-child {
  display: inline-block;
  width: 80%;
}

.eduRank>#classes>i {
  border-radius: 7px;
  font-style: normal;
  margin-left: 20px;
  text-align: center;
  display: inline-block;
  padding: 2px 5px;
}

.eduRank>#classes>i.ok {
  background: #0bb279;
  color: #FFF;
  border: #0bb279 1px solid;
  margin-bottom: 5px;
  font-size: 14px;
  font-weight: 600;
}

.eduRank>#classes>i.not {
  border: #bbb 1px solid;
  margin-bottom: 5px;
  color: #bbb;
  font-weight: 600;
  font-size: 14px;
}

div.week>div {
  flex-grow: 1;
}

div.week>div>div {
  flex-grow: 1;
  text-align: center;
  height: 30px;
  margin-bottom: 5px;
}

div.week>div.time>div {
  font-size: 16px;
}

div.okTime {
  background: #0bb279;
  border-radius: 15px;
  width: 25px;
  height: 25px;
  margin-left: auto;
  margin-right: auto;
}

div.notTime {
  width: 30px;
  height: 30px;
  margin-left: auto;
  margin-right: auto;
}

#grade > select {
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  border: none;
  background: #FFF;
  font-size: 17px;
}

#auth {
  padding-left: 15px;
  font-size: 17px;
  color: rgb(11, 178, 121);
}

#campusSelector {
  border: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  appearance: none;
  border: none;
  background: #FFF;
  outline: none;
  font-size: 18px;
  font-weight: 100;
}

</style>
<style>
#SelfIntro>textarea {
  border: none;
  padding: 0;
}
</style>
