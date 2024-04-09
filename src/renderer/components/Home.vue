<template>
  <div>
    
    <a-textarea @change="ckChanged()" v-model="ck" placeholder="Cookie" :rows="7"></a-textarea>
    <h3 style="text-align:center">姓名：{{name}}</h3>
    <a-input addon-before="deptIdEnc：" v-model="deptIdEnc"></a-input>
    <a-input addon-before="roomId：" v-model="roomId"></a-input>
    <a-button style="width: 100%" @click="test()" type="primary"
      >抢座测试</a-button
    >
    <a-textarea v-model="value" placeholder="状态日志输出" :rows="10" />
    <a-tag color="cyan">作者QQ9479421</a-tag>
  </div>
</template>
<script>
export default {
  data() {
    return {
      name:'待读取',
      ck: "lv=1; fid=484; _uid=148849340; UID=148849340; vc=2F8A2C4B542607FA569BE4D5BD01C4E9; xxtenc=4fdd5220d62fcdedd3f1823784c930ee; uf=d9387224d3a6095b57765ad6a6b62c3ae363fa9393f4406734405ff47a76e6a4661446e6b0d3fd03a61289cebd54745ec49d67c0c30ca5047c5a963e85f1109975617f401265aa5cce71fc6e59483dd36edeb94dbb42850875f2264a05ef362d3505cd7377821625; _d=1658638463707; vc2=C3B0A6E50D34AD342E9A16102C4386D5; vc3=WH00X3b448GWSUeHD337sVVKELVUyH4wfc%2FUq01IgdBdFHlWWJin23s%2BEyfni9a0Wykqluq1yg4BzCvaMpVU4t9JUjPuwE786%2B7dyyz2vrKa9rOpWJp9dzG3CJNhSDC2t%2B4RX05fXeZIdE3rwSg3MBXilk%2BcjY%2B8bWwVQeRXBWU%3D5df761a32fd32ed0e62db85ea36c610d; DSSTASH_LOG=C_38-UN_860-US_148849340-T_1658638463708; JSESSIONID=5334626F337D424F9157BFB06A57EB09.reserve_web_124; oa_deptid=484; oa_uid=148849340; oa_name=%E7%8E%8B%E5%9B%BD%E6%9D%83; oa_enc=9dfffae7e5d4303a0589665337afb222; route=3b84754f32aaf54a995f7d41f1a28848",
      
      deptIdEnc:'944acd736f0e58be',
      roomId:'7832',

      mornSeats: [],
      noonSeats: [],
      afterSeats: [],
      value: "",
      times: 0,
    };
  },
  mounted() {
    window.setInterval(() => {
      setTimeout(this.clock(), 0);
    }, 1000);
  },
  methods: {
    ckChanged(){
      this.name = decodeURIComponent(this.getCenterString(this.ck,'oa_name=',';'))
    },
    clock() {
      var d = new Date();
      var time = d.getHours() + ":" + d.getMinutes() + ":" + d.getSeconds();
      console.log(time);
      if (time == "19:0:0") {
        //重试次数清零
        this.times = 0;
        this.test();
      }
    },
    getCenterString(str, fromStr, endStr) {
      let from = str.indexOf(fromStr) + fromStr.length;
      if(from==-1){return '';}
      let end = str.indexOf(endStr, from);
      if(end==-1){return '';}
      return str.substring(from, end);
    },
    saveSeats(roomId, fromHour, endHour, day, phase) {
      let promise = new Promise((resolve, reject) => {
        this.$superagent
          .get(
            `https://office.chaoxing.com/data/apps/seat/getusedseatnums?roomId=${roomId}&startTime=${fromHour}%3A00&endTime=${endHour}%3A00&day=${day}`
          )
          .set("Cookie", this.ck)
          .end((err, res) => {
            console.log(res);
            var seatArr = [];
            //建立座位数组 一共60个座位
            for (let i = 0; i < 60; i++) {
              seatArr[i] = i + 1;
            }

            //读取已使用的座位数组
            var seatNums = res._body.data.seatNums;
            for (let i = 0; i < seatNums.length; i++) {
              //字符串整数001转整数1
              seatNums[i] = parseInt(seatNums[i]);
            }

            //判断座位是否已使用
            for (let i = 0; i < seatArr.length; i++) {
              let ele = seatArr[i];
              if (!seatNums.includes(ele)) {
                if (phase == 0) {
                  this.mornSeats.push(ele);
                } else if (phase == 1) {
                  this.noonSeats.push(ele);
                } else if (phase == 2) {
                  this.afterSeats.push(ele);
                }
              }
            }
            resolve();
          });
      });
      return promise;
    },
    test() {
      //清空早中晚数组
      this.mornSeats = [];
      this.noonSeats = [];
      this.afterSeats = [];
      //抢座房间号
      var roomId = this.roomId;
      //取次日日期 年月日
      var date = new Date();
      date.setDate(date.getDate() + 1);
      var day =
        date.getFullYear() + "-" + (date.getMonth() + 1) + "-" + date.getDate();

      Promise.all([
        //获取所有的可用座位，保存到数组中
        this.saveSeats(roomId, 7, 11, day, 0),
        this.saveSeats(roomId, 11, 15, day, 1),
        this.saveSeats(roomId, 15, 19, day, 2),
      ]).then((result) => {
        console.log("早上剩余座位", this.mornSeats);
        console.log("中午剩余座位", this.noonSeats);
        console.log("下午剩余座位", this.afterSeats);

        //判断全部同时都剩余的座位

        // this.mornSeats.push(41);
        // this.noonSeats.push(41);
        // this.afterSeats.push(41);

        let freeSeat = 0;
        for (let i = 0; i < this.mornSeats.length; i++) {
          if (
            this.noonSeats.includes(this.mornSeats[i]) &&
            this.afterSeats.includes(this.mornSeats[i])
          ) {
            freeSeat = this.mornSeats[i];
            //随机筛取
            if (Math.floor(Math.random() * 10) + 1 <= 3) {
              break;
            }
          }
        }
        if (freeSeat != 0) {
          if (freeSeat < 10) {
            freeSeat = "00" + freeSeat;
          } else {
            freeSeat = "0" + freeSeat;
          }
          //有座位 那么开抢
          console.log("符合要求的座位", freeSeat);
          this.$superagent
            .get(
              `https://office.chaoxing.com/front/third/apps/seat/list?deptIdEnc=${this.deptIdEnc}`
            )
            .set("Cookie", this.ck)
            .end((err, res) => {
              //读出pageToken fidEnc
              var pageToken = this.getCenterString(
                res.text,
                "pageToken=' + '",
                "'"
              );
              var fidEnc = this.getCenterString(res.text, "deptIdEnc = '", "'");
              console.log("fidEnc", fidEnc);
              console.log("pageToken", pageToken);

              this.$superagent
                .get(
                  `https://office.chaoxing.com/front/third/apps/seat/select?id=${roomId}&day=${day}&backLevel=2&pageToken=${pageToken}&fidEnc=${fidEnc}`
                )
                .set("Cookie", this.ck)
                .end((err, res) => {
                  //从中读出token
                  var token = this.getCenterString(res.text, "token: '", "'");
                  //抢上午 中午 下午
                  Promise.all([
                    this.robSeat(roomId, 7, 11, day, freeSeat, token, 0),
                    this.robSeat(roomId, 11, 15, day, freeSeat, token, 1),
                    this.robSeat(roomId, 15, 19, day, freeSeat, token, 2),
                  ]).then((result) => {
                    console.log(result);
                    if (!result.includes(false)) {
                      this.log("抢座成功，座位号：" + freeSeat);
                    } else {
                      this.log("抢座失败，座位号：" + freeSeat + "--" + result);
                      //如果没抢到，重试3次
                      if (this.times <= 2) {
                        this.times++;
                        this.log("重试" + this.times);
                        this.test();
                      }
                    }
                  });
                });
            });
        } else {
          //没有
          console.log("没有想要的座位");
          this.log("没有想要的座位");
        }
      });

      //抢座
    },
    robSeat(roomId, from, end, day, freeSeat, token, phase) {
      let promise = new Promise((resolve, reject) => {
        this.$superagent
          .get(
            `https://office.chaoxing.com/data/apps/seat/submit?roomId=${roomId}&startTime=${from}%3A00&endTime=${end}%3A00&day=${day}&seatNum=${freeSeat}&captcha=&token=${token}`
          )
          .set("Cookie", this.ck)
          .end((err, res) => {
            var time = "";
            switch (phase) {
              case 0:
                time = "上午";
                break;
              case 1:
                time = "中午";
                break;
              case 2:
                time = "下午";
                break;
            }
            if (res._body.success) {
              console.log(time, "预订成功");
              this.log("预订成功");
              resolve(true);
            } else {
              console.log(time, "预订失败", res._body);
              this.log("预订失败" + JSON.stringify(res._body));
              resolve(false);
            }
          });
      });
      return promise;
    },
    log(str) {
      this.value += new Date().toLocaleString() + " --- " + str + "\n";
    },
  },
};
</script>
