<template>
  <div class="order-container">
    <div class="order-content">
      <div class="order-section">
        <el-row class="order-head">
          <el-col :span="4">
            <el-button type="primary" icon="arrow-left" @click="toBack()"></el-button>
          </el-col>
          <el-col :span="16" class="head">{{prompts}}</el-col>
        </el-row>
      </div>
    </div>

    <div v-show="showTeacher" class="contents">

      <div style="justify-content: space-between;" id="Main" class="Content1">
        <div id="Meta">
          <div id="Name" style="display:flex;">
            <div style="outline:none;font-weight: 800;font-size: 24px;color: rgb(11, 178, 121);width: 95px;border: none;">{{orderedOne.name}}</div>
            <div v-if="orderedOne.sex" style="display: inline-block;position:relative;top:4px;margin-left: 5px;"><img width="20" src="../assets/female.svg"></div>
            <div v-else style="display: inline-block;position:relative;top:4px;margin-left: 5px;"><img width="20" src="../assets/male.svg"></div>
          </div>

          <div id="DetailMeta" style="margin-top: 20px;">
            <!-- <div id="edu" style="outline:none;color: rgb(187, 187, 187);margin-right:5px;font-weight:300;">{{edu[orderedOne.edu]}}</div>
            <div id="edu" style="outline:none;margin-right:5px;font-weight:300;">{{grades[orderedOne.edu][orderedOne.grade]}}</div> -->
            <div v-if="orderedOne.auth" id="auth">
              <img width="20" style="padding-top:3px;" src="../assets/auth.svg">
            </div>
            <div v-else id="auth">
              未认证
            </div>
          </div>
        </div>

        <div id="rate" style="text-align: right;font-size: 23px;color: #0bb279; font-weight: 600;">
          <div style="color: #000;font-weight: 300;font-size:16px;padding-top: 5px;">
            注册“猴啊”
            <span style="font-weight: 400;color:rgb(11, 178, 121)">{{Math.floor(((new Date()) - orderedOne.joinedAt)/3600000/24)}}天</span>
          </div>
          <div style="margin-top:15px;letter-spacing:2px;">
            <!-- {{orderedOne.rate}}
            <i style="font-size: 12px;font-style: normal; font-weight: 400;"> 分</i> -->
          </div>
        </div>
      </div>

      <div id="detail">
        <div class="Content1">
          <div>薪资</div>
          <div>
            <span style="font-size: 17px;margin-right: 5px;color: #0bb279;font-weight: 600;">¥ {{orderedOne.salary}}</span>
            <span style="font-size:16px; font-weight:200;">/小时</span>
          </div>
        </div>

        <div class="Content1">
          <div>地点</div>
          <div id="editplace" style="display:flex; justify-content: flex-start;flex-wrap: nowrap">
            <!--TODO:store中新建数据存储信息-->
            <!-- <div style="font-size:16px; font-weight:200;text-align:right;">请确认地点:</div> -->
            <div v-on:change="preventWindow" style="outline:none;font-weight: 600;font-size: 16px;color: rgb(11, 178, 121);width: 60px;border: none;text-align:right;">{{orderedOne.campus.name}}</div>
          </div>
        </div>

        <!-- <div class="Content1">
          <div>年级</div>
          <el-select size="small" v-model="orderedOne.edu" :placeholder="edu[orderedOne.edu]">
            <el-option v-for="(item,index) in edu" :key="index" :label="item" :value="index">
            </el-option>
          </el-select>
        </div> -->
        <div class="Content1" style="flex-direction: column;">
          <div>科目</div>
          <div id="class">
            <div class="eduRank" v-for="(subjects, i) in orderedOne.subjectTable">
              <div>{{subjects.name}}</div>
              <div id="classes">
                <i v-for="(subject, j) in subjects.subjects" v-bind:class="subject.checked?'ok':'not'">
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
          <div v-for="i in [0,1,2,3,4,5,6]" :key="i">
            <div>
              {{days[i]}}
            </div>
            <div v-for="j in [0,1,2]" :key="j" @click="toggleOrderCalendar([i,j])">
              <div class=" time " v-bind:class="orderedOne.availableTime[i][j]? 'okTime': 'notTime' ">
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>

    <div v-show="showPay " class="contents ">
      <div class="pay ">
        <div>
          <span>本次交易，猴啊家教</span>
          <span style="color:#33BF8F ">不收取任何费用</span>
        </div>
        <div>
          <span style="color:#888;font-size:16px ">¥ </span>
          <span style="color:#33BF8F;font-size:40px ">0</span>
        </div>
      </div>
    </div>

    <div v-show="showDone " class="contents ">
      <div class="Done ">
        <div class="Cong ">恭喜您！订单已经生效！</div>
        <div class="Notes ">
          <p>您可以在
            <span style="color:#33BF8F;font-weight:500 ">订单管理界面</span>管理和查看本订单。</p>
          <p>我们已为您通知了对方家长，您应该在近期接到家长的联系，
            <span style="color:#33BF8F;font-weight:500 ">请留意您登记在猴啊的联系方式。</span>
          </p>
        </div>
      </div>
    </div>

    <div class="next ">
      <el-button @click="gonext() " size="large " style="width:6em; " type="primary ">{{buttonText}}</el-button>
    </div>
  </div>
</template>

<script>
  import {
    mapGetters,
    mapMutations
  } from 'vuex'
  // import AV from 'leancloud-storage' // 数据库对象
  export default {
    data() {
      return {
        showTeacher: true,
        showPay: false,
        showDone: false,
        prompts: '正 在 生 成 订 单',
        buttonText: '下一步',
        // edu: ['小学', '初中', '高中', '本科'],
        // grades: [['一年级', '二年级', '三年级', '四年级', '五年级', '六年级'], ['初一', '初二', '初三'], ['高一', '高二', '高三'], ['大一', '大二', '大三', '大四']],
        // classes: [['全科', '陪读'], ['数学', '科学', '英语'], ['数学', '理综', '英语']],
        days: ['一', '二', '三', '四', '五', '六', '日'],
        rawUser: ''
      }
    },
    computed: {
      ...mapGetters([
        'orderedOne'
      ])
    },
    mounted: function () {
      // this.rawUser = this.orderedOne
    },
    methods: {
      ...mapMutations([
        'toggleOrderTeach',
        'toggleOrderCalendar'
      ]),
      toBack: function () {
        if (this.showTeacher) {
          this.showTeacher = false
          this.$router.push('/reserve')
          this.prompts = '正 在 生 成 订 单'
          console.log(this.orderedOne)
        } else if (this.showPay) {
          this.showPay = false
          this.showTeacher = true
        } else if (this.showDone) {
          this.showDone = false
          this.showPay = true
          this.prompts = '正 在 准 备 付 款'
        }
      },
      getClass: function () {
        switch (this.orderedOne.grade) {
          case 0:
            return 0
          case 1:
            return 1
          case 2:
            return 2
        }
      },
      gonext: function () {
        let self = this
        if (this.showTeacher) {
          this.showTeacher = false
          this.showPay = true
          this.showDone = false
          this.prompts = '正 在 准 备 付 款'
          console.log(this.orderedOne)
        } else if (this.showPay) {
          // 预约的代码，与后端交互
          // const teacherMapUser = new AV.Object('TeacherMapUser')
          // const targetUser = new AV.Object.createWithoutData('TeacherList', self.orderedOne.objectId)
          // teacherMapUser.set('Teacher', targetUser)
          // teacherMapUser.set('User', AV.User.current())
          // teacherMapUser.set('status', '未查看')
          // teacherMapUser.set('salary', self.orderedOne.salary)
          // teacherMapUser.set('campus', self.orderedOne.campus)
          // teacherMapUser.save()
          fetch('https://api.houaa.xyz/order/create/', {
            method: 'POST',
            credentials: 'include',
            body: JSON.stringify({
              teacherId: self.orderedOne.teacherId,
              studentId: self.orderedOne.studentId,
              subjectTable: self.orderedOne.subjectTable,
              timeTable: self.orderedOne.availableTime
            })
          }).then(raw => raw.json())
          .then(json => {
            if (json.status === 'error') {
              self.$message(json.payload)
            } else {
              self.$message('预约成功！')
              self.prompts = '完 成'
              self.showTeacher = false
              self.showPay = false
              self.showDone = true
              self.buttonText = '去查看'
              // console.log(this.orderedOne)
            }
          })
        } else if (this.showDone) {
          this.$router.push('/reserve')
          this.showTeacher = false
          this.showPay = false
          this.showDone = false
          // console.log(this.orderedOne)
        }
      },
      preventWindow() {
        window.preventWindowClose = true
      }
    }
  }

</script>

<style>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity .3s
  }

  .fade-enter,
  .fade-leave-to {
    opacity: 0
  }

</style>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .order-container::-webkit-scrollbar {
    display: none
  }

  .order-container {
    overflow-x: hidden;
    overflow-y: scroll;
    height: 100%;
    width: 90%;
    margin-left: auto;
    margin-right: auto;
    color: #7e7e7e;
  }

  .head {
    text-align: center;
    font-size: 18px;
    font-weight: 500;
    color: #000;
  }

  .order-head {
    height: 60px;
    display: -webkit-flex;
    display: flex;
    -webkit-align-items: center;
    align-items: center;
    -webkit-justify-content: center;
    justify-content: center;
    border-bottom: 1px solid rgb(238, 238, 238);
  }

  .contents {
    margin: 0 10px;
  }

  .Content1 {
    display: flex;
    justify-content: space-between;
    border-bottom: 1px solid rgb(238, 238, 238);
    padding-bottom: 20px;
    padding-top: 20px;
  }

  .Content1>:first-child {
    font-size: 1.2em;
    font-weight: 600;
    color: #000;
  }

  .next {
    margin: 20px 0 30px 0;
    text-align: center;
  }

  .pay {
    margin-top: 200px;
    text-align: center;
  }

  .pay>:first-child {
    margin-bottom: 20px;
    font-size: 16px;
    font-weight: 400;
    color: #888;
  }

  .cost {
    display: block;
    font-size: 40px;
  }

  .Cong {
    margin-top: 150px;
    margin-bottom: 100px;
    font-size: 20px;
    color: #000;
    text-align: center;
  }

  .Notes {
    text-align: center;
    margin: 15px;
    font-size: 13px;
  }

  #DetailMeta {
    display: flex;
    flex-direction: row;
  }

  div.week {
    margin-top: 20px;
    display: flex;
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
    width: 30px;
    height: 30px;
    margin-left: auto;
    margin-right: auto;
  }

  div.notTime {
    width: 30px;
    height: 30px;
    margin-left: auto;
    margin-right: auto;
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

</style>
