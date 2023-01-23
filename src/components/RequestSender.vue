<template>
  <select class="method-dropdown" v-model="method">
    <option value="GET">GET</option>
    <option value="POST">POST</option>
    <option value="PUT">PUT</option>
    <option value="DELETE">DELETE</option>
  </select>
  <input class="url-input" v-model="url" placeholder="Enter request url"/>
  <button class="send-button" @click="sendRequest()">Send</button>

  <div class="request-portion">
    <div class="request-tabs">
      <button class="tablinks active" @click="openTab($event, 'Authorization')">Authorization</button>
      <button class="tablinks" @click="openTab($event, 'Headers')">Headers <span class="primary-text">{{ '(' + headerUsedCount + ')'}}</span></button>
      <button class="tablinks" @click="openTab($event, 'Body')">Body</button>
      <button class="tablinks" @click="openTab($event, 'Params')">Params</button>
    </div>

  <div id="Authorization" class="tabcontent">
    <div class="left">
      <label>Type</label>
      <select class="auth-dropdown" v-model="authType">
        <option value="" selected>None</option>
      <option value="Basic">Basic</option>
      <option value="Bearer">Bearer</option>
    </select>
    </div>
    <div class="right">
         <div v-if="authType==='Basic'" class="basic">
              <label for="username">Username</label>
              <input type="text" id="username" placeholder="eg: Tom" v-model="username">
              <label for="password">Password</label>
              <input type="password" id="password" placeholder="" v-model="password">
         </div>
         <div v-if="authType==='Bearer'" class="bearer">
          <label for="bearer">Bearer Token</label>
          <input type="text" id="bearer" placeholder="eg : sbakshksasYSHSALXNXXsnssn===" v-model="bearerToken">
         </div>
         <div class="no-auth" v-if="authType===''">
            <p>No authorization is required.</p>
         </div>
    </div>
    
  </div>
  <div id="Params" class="tabcontent">
    <table class="table">
      <!-- <button @click="addHeader">Add</button>
      <button @click="removeHeader">Remove</button> -->

      <tr class="header-row">
        <th class="checkbox-cell"></th>
        <th>Key</th>
        <th>Value</th>
      </tr>
      <tr v-for="(item, index) in queryParms" :key="item">
        <td class="checkbox-cell">
          <input type="checkbox" name="" :id="'check-' + index" :checked=item.isNeeded @change="isQueryParamNeeded($event,index)">
        </td>
        <td>
          <input type="text" name="" :id="'key-' + index" :value=item.key @focusout="addQueryKey($event,index)">
        </td>
        <td>
          <input type="text" name="" :id="'value-' + index" :value="item.value" @focusout="addQueryValue($event, index)" >
        </td>
      </tr>
    </table>
    
  </div>

  <div id="Headers" class="tabcontent">
    <table class="table">
      <!-- <button @click="addHeader">Add</button>
      <button @click="removeHeader">Remove</button> -->

      <tr class="header-row">
        <th class="checkbox-cell"></th>
        <th>Key</th>
        <th>Value</th>
        <th>Description</th>
      </tr>
      <tr v-for="(item, index) in headers" :key="item">
        <td class="checkbox-cell">
          <input type="checkbox" name="" :id="'check-' + index" :checked=item.isNeeded @change="isNeeded($event,index)">
        </td>
        <td>
          <input type="text" name="" :id="'key-' + index" :value=item.key @focusout="addKey($event,index)">
        </td>
        <td>
          <input type="text" name="" :id="'value-' + index" :value="item.value" @focusout="addValue($event, index)" >
        </td>
        <td>
          <input type="text" name="" :id="'description-' + index" :value="item.description" @focusout="addDescription($event, index)" >
        </td>
      </tr>
    </table>
  </div>


  <div id="Body" class="tabcontent">
    <div id="jsoneditor-request-body" style="width: 100%; height: 100%;"></div>
  </div>

  </div>

  <div class="response-portion">
    <div class="response-header-details">
      <h3>Response</h3>
      <div class="response-details">
        <span v-if="status" class="el">Status: <span class="primary-text">{{ status }}</span></span>
        <span v-if="time" class="el">Time: <span class="primary-text">{{ time }}</span></span>
      </div>
    </div>
<pre  class="response-body"> 
  <div class="preview">
    <img src="../assets/send.jpeg" alt="">
    <p>Send us request, sit and relax</p>
  </div>
</pre>
  </div>
</template>

<script>
import JSONEditor from 'jsoneditor/dist/jsoneditor.min.js'
import HttpStatusEnum from './status.js'
export default {
  name: 'RequestSender',
  data() {
    return {
      headers: [
        { key: "", value: "" , isNeeded: false, description : ""}
      ],
      authorization: "",
      body: null,
      url : "https://jsonplaceholder.typicode.com/todos",
      method : "GET",
      response : "",
      status : "",
      time : "",
      authType : "",
      username : "",
      password : "",
      bearerToken  : "",
      headerUsedCount : 0,
      queryParms : [{ key: "", value: "" , isNeeded: false}]
    }
  },
  props: {

  },
  methods: {
    openTab(evt, tab) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent");
      for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
      }
      tablinks = document.getElementsByClassName("tablinks");
      for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
      }
      document.getElementById(tab).style.display = "flex";
      evt.currentTarget.className += " active";
    },
    addHeader() {
      this.headers = [...this.headers, { key: "", value: "" , isNeeded:false, description:""}]
    },
    addQueryParam() {
      this.queryParms = [...this.queryParms, { key: "", value: "" , isNeeded:false}]
    },
    removeHeader() {
      this.headers = this.headers.slice(0, this.headers.length - 1)
    },
    async sendRequest() {
      let authorization = this.getAuthorization();
      if(authorization) this.headers = [...this.headers, { key: "Authorization", value: authorization }]
      let headersGiven = this.getHeaders();
      let time1 = performance.now();
      const res = await fetch(this.url, {
          method: this.method,
          headers: headersGiven,
          body: this.body?this.body:null,
        });

        this.response = await res.json();
        let existingDiv = document.querySelector("#jsoneditor-response-body")
        let container = null
        if(existingDiv) {
          document.querySelector(".response-body").removeChild(existingDiv)
        } 
        container = document.createElement("div");
        container.id =  "jsoneditor-response-body";
        container.style.marginTop = "-30px"
        document.querySelector(".preview").style.display = 'none'
        document.querySelector(".response-body").appendChild(container);
        const options = {}
        const editor = new JSONEditor(container, options);
        editor.set(this.response)
        editor.setMode("preview");
        
        console.log(HttpStatusEnum);
        this.status = res.status  
        let time2 = performance.now();
        this.time =  (time2 - time1).toFixed(2) + "ms";
    },
    getHeaders() {
        let headers = {};
        this.headers.forEach((el) => {
          if(el.isNeeded)
            headers = {...headers, [el.key] : el.value}
        })
        return headers
    },
    getAuthorization() {
       let authorization = "";
       switch(this.authType) {
          case "Basic" :
            authorization = `Basic ${window.btoa(unescape(encodeURIComponent(this.username + ':' + this.password)))}`
            break;
          case "Bearer" :
            authorization = `Bearer ${this.bearerToken}`;
            break;
       }
       return authorization;
    },
    addKey(event, index) {
      this.headers[index] = {...this.headers[index], "key" : event.target.value};
      this.checkIfHeaderNeededAndAdd();
    },

    addValue(event, index) {
      this.headers[index] = {...this.headers[index], "value" : event.target.value};
      this.checkIfHeaderNeededAndAdd();
    },

    isNeeded(event, index) {
      let isNeeded  = !this.queryParms[index].isNeeded;
      this.queryParms[index] = {...this.queryParms[index], "isNeeded" : isNeeded}      
    },

    addDescription(event, index) {
      this.headers[index] = {...this.headers[index], "description" : event.target.value}
    },

    addQueryKey(event, index) {
      this.queryParms[index] = {...this.queryParms[index], "key" : event.target.value};
      this.checkIfQueryParamNeededAndAdd();
      this.formQueryString()
    },

    formQueryString() {
        let queryString = "?"
        this.queryParms.forEach((param) => {
           if(param.key!=="" || param.value!=="")
              queryString = queryString.concat(param.key.trim(), "=",param.value.trim(), "&")
        })
        if(queryString.charAt(queryString.length-1) === '?' || queryString.charAt(queryString.length-1) === '&')
          queryString = queryString.substring(0, queryString.length-1);
          console.log(queryString);
        this.url = this.url.split("?")[0].concat(queryString)
    },

    addQueryValue(event, index) {
      this.queryParms[index] = {...this.queryParms[index], "value" : event.target.value};
      this.checkIfQueryParamNeededAndAdd();
      this.formQueryString()
    },

    isQueryParamNeeded(event, index) {
      let isNeeded  = !this.queryParms[index].isNeeded;
      this.queryParms[index] = {...this.queryParms[index], "isNeeded" : isNeeded}      
    },
    checkIfQueryParamNeededAndAdd() {
      let lastEl = this.queryParms[this.queryParms.length-1];
      if(lastEl.key !== "" || lastEl.value !== "") this.addQueryParam();
    },
  
    checkIfHeaderNeededAndAdd() {
      let lastEl = this.headers[this.headers.length-1];
      if(lastEl.key !== "" || lastEl.value !== "") this.addHeader();
    }

  },
  mounted() {
    try{
      const container = document.getElementById("jsoneditor-request-body");
      const options = {}
      const editor = new JSONEditor(container, options);
      editor.set('')
      editor.setMode("code")
    } catch (err) {
      console.error(err);
    }
  }
}
</script>

<style scoped>
@import url("jsoneditor/dist/jsoneditor.min.css");
.method-dropdown {
  padding: 15px 15px;
  background-color: #f2f2f2;
  border: 1px solid darkgray;
  border-radius: 3px;
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
  font-weight: 600;
}

.url-input {
  padding: 16px;
  border: 1px solid darkgray;
  border-left: 0;
  background-color: #f2f2f2;
  outline: none;
  width: 60%;
  border-radius: 5px;
  border-top-left-radius: 0;
  border-bottom-left-radius: 0;
}

.url-input:focus {
  background-color: white;
}

.send-button {
  margin-left: 20px;
  padding: 16px 20px;
  border: 1px solid #3883fa;
  background-color: #3883fa;
  color: white;
  font-weight: 600;
  border-radius: 5px;
  cursor: pointer;
}

.request-portion {
  margin-top: 2%;
  height: 35vh;
}
.request-tabs {
  overflow: hidden;
  border: 1px solid #ccc;
  background-color: #f1f1f1;
}

.request-tabs button {
  background-color: inherit;
  float: left;
  border: none;
  outline: none;
  cursor: pointer;
  padding: 14px 16px;
  transition: 0.3s;
  font-size: 12px;
}

.request-tabs button.active {
  border-bottom: 4px solid #3883fa;
}

.tabcontent {
  padding-top: 1%;
  padding-bottom: 1%;
  display: none;
  border: none;
}
.checkbox-cell{
  min-width:  none !important;
  max-width: 20px !important;
}

th,
td {
  border: 1px solid darkgray;
  padding: 5px;
}

.w-20 {
  width: 20%;
}

.w-30 {
  width: 30%;
}

td {
  padding: 3px;
}

td>input {
  height: 100%;
  border: none;
  width: 90%;
  outline: none;
  padding: 10px;
}

table {
  border-spacing: 0;
  width: 100%;
}

.jsoneditor-menu {
  display: none !important;
}

.response-portion {
   border-top: 1px solid lightgray;
}

.response-portion  pre {
  max-height: 60vh;
   overflow: auto;
}

.primary-text {
  color: lightseagreen;
}

.response-header-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.response-header-details .el {
   margin: 0 10px;
}

#Authorization {
  display: flex;
  justify-content: space-between;
  padding: 1%;
  height: 75%;
}

#Authorization > .left , #Authorization > .right {
  width: 50%;
  padding: 1% 2%;
}

.basic , .bearer{
  display: flex;
  flex-direction: column;
}

label  {
  margin: 5px 0;
  font-weight: 900;
}

.left > select {
  padding: 15px 15px;
  background-color: rgb(234, 234, 234);
  border: 1px solid darkgray;
  border-radius: 3px;
  max-width: 200px;
}

.left {
  border-right: 1px solid darkgray;
  display: flex;
  flex-direction: column;
}

input {
  padding: 10px 15px;
}

h3 {
  margin-block-start: 0.5em !important;
  margin-block-end: 0.5em !important;

}

pre {
  margin: 0;
}

input[type="checkbox" i] {
  height: 20px;
}

.preview  {
  justify-content: center;
  align-items: center;
  display: flex;
  flex-direction: column;
  color: #3883fa;
}


</style>
