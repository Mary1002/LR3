<template>
  <div id="app">
    <div class="text-center">
      <p></p>
      <button v-on:click="to_main" type="button" class="btn btn-dark btn-lg ">На головну</button>
      <p></p>
    </div>
    
    <div v-if="status == 'main'" class="text-center">
      <button v-on:click="to_append_driver" type="button" class="btn btn-primary btn-lg">Зареєструватися як водій</button>
      <button v-on:click="to_append_driver_mark" type="button" class="btn btn-primary btn-lg">Поставити відгук водієві</button>
      <button v-on:click="to_list_drivers" type="button" class="btn btn-primary btn-lg">Рейтинг водіїв</button>
      <button v-on:click="to_find_driver" type="button" class="btn btn-primary btn-lg">Інформація про водія</button>
    </div>
    <div v-else-if="status == 'append_driver'" class="align-items-center container" style="width: 700px">
      <form onsubmit="return false;" class="form-control text-center "> 
        <div class="form-group">
          <input class="form-control"  type="text" v-model="append_driver_data[0]" placeholder="ПІБ" ref="pib">
          <small  class="form-text text-muted">ПІБ</small>
          <input class="form-control" type="email" v-model="append_driver_data[1]" placeholder="email">
          <small  class="form-text text-muted">Ваша поштова скринька</small>
          <input class="form-control" type="number" v-model="append_driver_data[2]" placeholder="phone">
          <small  class="form-text text-muted">Ваш робочий номер телефону</small>
          <textarea class="form-control" type="text" v-model="append_driver_data[3]" cols="20" rows="5" placeholder="інфо"></textarea>
          <small  class="form-text text-muted">Інформація про вас</small>
        </div>
        
        <input v-on:click="send_data" type="submit" class="btn btn-primary" value="Зареєструватися">
      </form>
    </div>
    <div v-else-if="status == 'append_driver_mark'" class="align-items-center container" style="width: 700px">
      <form onsubmit="return false;" class="form-control text-center "> 
        <div class="form-group">
          <select v-model="select_driver" class="form-select form-select-sm"> 
            <option value="" disabled selected>Виберіть водія</option>
            <option v-for="item in drivers_arr" :key="item" v-bind:value="item">
              {{item}}
            </option>
          </select>
          <p></p>
          <input type="range" class="form-control-range form-range" min=0 max=10 step=1 v-model="you_mark" >
            {{you_mark}}
          <p></p>
          <small  class="form-text text-muted">Ваша оцінка</small>
          <textarea class="form-control" ref="you_resp" cols="20" rows="5" placeholder="Відгук" v-model="you_text"></textarea>
          <small  class="form-text text-muted">Ваш відгук</small>
          <input  v-on:click="send_response" type="submit" value="Відправити" class="btn btn-primary">
        </div>
      </form>
      
      
    </div>
    <div v-else-if="status == 'list_drivers'" class="align-items-center container" style="width: 700px">
      <ul v-if="list_drivers" class="list-group list-group-flush">
        <li v-for="item in list_drivers" :key="item" class="list-group-item">
          {{item.star.toFixed(2)}} &#10027;, {{item.pib}}, &#9742; - {{item.phone}}, &#128386; -<a v-bind:href="'mailto:'+item.email">{{item.email}}</a> , {{item.info}}
        </li>
      </ul>
    </div>
    <div v-else-if="status == 'find_driver'" class="align-items-center container" style="width: 700px">
       <select v-model="select_driver" class="form-select form-select-sm"> 
            <option value="" disabled selected>Виберіть водія</option>
            <option v-for="item in drivers_arr" :key="item" v-bind:value="item">
              {{item}}
            </option>
        </select>
        <p></p>
      <button v-on:click="get_driver_info" class="btn btn-primary">отримати інформацію</button>
      <p></p>
      <div v-if="one_driver != null">
        <li class="list-group-item">
          {{one_driver.pib}}, &#128386; -<a v-bind:href="'mailto:'+one_driver.email">{{one_driver.email}}</a>, &#9742; - {{one_driver.phone}}, {{one_driver.info}}
        </li>
        <ul class="list-group list-group-flush">
          <li v-for="item in one_driver.marks.length" :key="item" class="list-group-item">
            {{one_driver.dates[item-1]}}, {{one_driver.marks[item-1]}} &#10027;, {{one_driver.resps[item-1]}}
          </li>
        </ul>
      </div>
    </div>
    <div v-else>
      error!
    </div>
  <!-- footer-->
  <footer class="footer bg-light text-center text-lg-start">
  <!-- Copyright -->
  <div class="text-center p-3" style="background-color: rgba(0, 0, 0, 0.2);">
    © 2021 Copyright: Маша
  </div>
  <!-- Copyright -->
</footer>
  <!-- footer-->
  </div>
</template>
<!--

  status = main, append_driver, append_driver_mark, list_drivers, find_driver
  
 -->
<script>
import axios from 'axios'
function compare( a, b ) {
  if ( a.star > b.star ){
    return -1;
  }
  if ( a.star < b.star ){
    return 1;
  }
  return 0;
}
export default {
  name: 'app',
  data () {
    return {
      server: 'http://localhost:3000/',
      status: "main",
      append_driver_data: Array(4), // дані відправки форми
      drivers_arr: Array(),  // список водіїв для вибору зі списку)
      list_drivers: Array(), // рейтинг
      select_driver: '', // вибраний водій
      you_mark: 5, // оцінка водія
      you_text: '', // ваш відгук
      one_driver: null
    }
  },
  methods:{
    to_main(){
      this.status = "main";
      this.select_driver = '';
      this.you_text = '';
      this.you_mark = 5;
      this.drivers_arr = new Array();
      this.one_driver = null;
    },
    to_find_driver(){
      axios.get(this.server+'drivers').then((response) => { 
        for(let i=0;i<response.data.length;i++){
          this.drivers_arr.push(response.data[i]); 
        }
      },err => alert('server eror'));
      this.status = "find_driver";
    },
    get_driver_info(){
      axios.post(this.server+'find_one_drivers', {
        pib: this.select_driver
      }).then((response) => { 
        this.one_driver = response.data;
      },err => alert('server eror'));
    },
    to_append_driver(){
      this.status = "append_driver";
    },
    to_append_driver_mark(){
      axios.get(this.server+'drivers').then((response) => { 
        for(let i=0;i<response.data.length;i++){
          this.drivers_arr.push(response.data[i]); 
        }
      },err => alert('server eror'));
      this.status = "append_driver_mark";
    },
    to_list_drivers(){
      axios.get(this.server+'drivers_list').then((response) => { 
        this.list_drivers =  response.data;
        this.list_drivers.sort(compare);
      },err => alert('server eror'));
      this.status = "list_drivers";
    },
    send_data(){
      if(this.append_driver_data[0] != null && this.append_driver_data[1] != null && this.append_driver_data[2] != null && this.append_driver_data[3] != null){
        axios.post(this.server+'drivers', {
          pib: this.append_driver_data[0],
          email: this.append_driver_data[1],
          phone: this.append_driver_data[2],
          info: this.append_driver_data[3]
        }).then((response) => {
            console.log("OK");
          }, (error) => {
            console.log(error);
        },err => alert('server eror'));  
        this.append_driver_data = new Array(4);
      }
      else{
        this.$refs.pib.focus();
      }
    },
    send_response(){
      if(this.select_driver != '' &&  this.you_text != '' ){
        axios.post(this.server+'drivers_resp', {
          driver: this.select_driver,
          mark: this.you_mark,
          text: this.you_text
        }).then((response) => {
            console.log("OK");
          },err => alert('server eror'));  
        this.you_text = '';
      }
      else{
        this.$refs.you_resp.focus();
      }
    }
  }
}
</script>

<style>
body{
  background-color: #785e5e;
background-image: url("data:image/svg+xml,%3Csvg width='30' height='30' viewBox='0 0 30 30' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M0 10h10v10H0V10zM10 0h10v10H10V0z' fill='%23e7cb67' fill-opacity='0.79' fill-rule='evenodd'/%3E%3C/svg%3E");
}
.footer {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
}
</style>
