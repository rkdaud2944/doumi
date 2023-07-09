<template>
    <div class="box">
      <img class="Home_img" src = "@/assets/Home.png" />
      <img class="Arrow_img" src = "@/assets/Arrow-right.png" />
      <a class="text">교육신청</a>
      <p style="
        text-align: right;
        font-style: oblique;
        font-size: 16px;
        font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;">인천미래채움</p>
    </div>
    <div class="box2">
      <a class="text2">교육 신청</a>
    </div>
</template>

<style scoped>

.box {
  display: block;
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  width: 80%;
  background-color: #ffffff;
  margin-top: 40px;
  border-bottom: solid 1px #8f8f8f;
  height: 50px;
}

.box2 {
  display: block;
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  width: 80%;
  background-color: #ffffff;
  margin-top: 40px;
  border-bottom: solid 2.5px #000000;
  height: 40px
}
.Home_img {
  display: block;
  margin-left: auto;
  margin-right: auto;
  margin: 0px;
  width: 30px;
  text-align: left;
  float: left;
}

.Arrow_img {
  margin-left: 20px;
  margin-top: 5.5px;
  float: left;
  width: 12px;
}

.text {
  margin-left: 20px;
  float: left;
  margin-top: 5px;
  color: black;
}

.text2 {
  margin-left: 10px;
  float: left;
  margin-top: 2px;
  font-size: 22px;
  font-weight: bold;
  color: black;
}

</style>
