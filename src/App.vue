<template>
  <div id="app">
    <div id="content">
      <h1>Zug Botellón</h1>
      <h2>Gleis Randomizer</h2>
      <div>
        <input placeholder="Startbahnhof eingeben" type="text" v-model="station" />
      </div>
      <button :disabled="!station" @click="getConnections()">Gleis würfeln</button>
      <transition name="slide-fade" mode="out-in"></transition>

      <div id="border">
        <div id="tafel">
          <div v-if="connection">
            <div class="nach">{{connection.to}}</div>
            <div class="zeit">{{connection.passList[0].departure | moment('HH:mm')}}</div>
            <div class="gleis">
              <span class="text">Gleis</span>
              <span class="zahl">{{connection.passList[0].platform}}</span>
            </div>
          </div>
          <div v-if="loading">
            <img src="./assets/rauschen.gif">
          </div>
        </div>
      </div>
      <button v-if="connection" @click="randomizeStops()">Ausstieg würfeln</button>
      <div v-if="connection && stopIndex">
        <ul>
          <template v-for="(pass,index) in connection.passList">
          <li  v-if="index != 0" :key="pass.station.id">
            <div class="pass" :class="{stop:index == stopIndex}">
              {{pass.station.name}}
            </div>
           
          </li>
           </template>
        </ul>
      </div>
      <div v-if="pass">Austeigen: {{pass.station.name}}</div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data: () => ({
    connections: null,
    connection: null,
    loading: true,
    stopIndex: null,
    station: null,
  }),
  name: "app",
  mounted() {},
  methods: {
    async getConnections() {
      this.stopIndex = null
      this.connections = await axios.get(
        "https://transport.opendata.ch/v1/stationboard",
        {
          params: { "transportations[]": "train", station: this.station, limit: 20 }
        }
      );
      let rndm = Math.floor(
        Math.random() * this.connections.data.stationboard.length
      );
      this.connection = this.connections.data.stationboard[rndm];
    },
    randomizeStops() {
      this.stopIndex = Math.floor(
        Math.random() * this.connection.passList.length
      );
      if(this.stopIndex == 0) this.stopIndex= this.connection.passList.length -1
    }
  }
};
</script>

<style>



body {
  font-family: Arial, Helvetica, sans-serif;
  margin: 0;
}

input{
  width: calc(100% - 10px);
  padding: 5px;
  font-size: 20px;
  margin-top: 15px;
}

button{
  font-size: 20px;
  padding: 20px;
  margin-bottom: 15px;
  
  margin-top: 15px;
}

h1,
h2 {
  margin: 0;
}

#content {
  min-width: 100px;
  max-width: 700px;
  box-sizing: border-box;
  margin: 0 auto;
  padding: 15px;

  border: 1px solid gray;
  min-height: 100vh;
}

#border {
  background-color: rgb(7, 14, 109);
  height: 75px;
  padding: 20px;
  position: relative;
}

#tafel {
  box-sizing: border-box;
  padding: 5px;
  background-color: rgb(26, 26, 189);
  border-radius: 4px;
  height: 100%;
  color: rgb(222, 233, 243);
  font-weight: bold;
  position: relative;
}

#tafel img {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.1;
  width: 100%;
  height: 100%;
}

.gleis {
  position: absolute;
  top: 5px;
  right: 5px;
  border: 2px solid #fff;
  border-radius: 4px;
  min-width: 50px;
  height: 50px;
  padding: 5px;
  text-align: center;
}

.zeit {
  position: absolute;
  top: 5px;
  left: 5px;
}

.nach {
  position: absolute;
  left: 5px;
  bottom: 5px;
  font-size: 30px;
}

.gleis .text {
  font-size: 10px;
  display: block;
  text-align: center;
}
.gleis .zahl {
  display: block;
  font-size: 30px;
}

.stop{
  color: red;
  font-size: 30px!important;
}

ul{list-style: none}

.pass{
  font-size: 20px;
}

</style>
