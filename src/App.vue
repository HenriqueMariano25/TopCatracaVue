<template>
  <div id="app">
    <div class="contagem">
      <div class="formulario_contagem">
        <h2>Contagens de passagem pelo refeitorio</h2>
        <form @submit.prevent="receberContadores()">
          <label for="data_contagem">Data</label>
          <input id="data_contagem" type="date" name="data_contagem" v-model="contadores.data"/>

          <label for="hora_inicial_contagem">Hora Inicial</label>
          <input id="hora_inicial_contagem" type="time" name="hora_inicial_contagem" v-model="contadores.horaInicial"/>

          <label for="hora-final-contagem">Hora Final</label>
          <input id="hora-final-contagem" type="time" name="hora-fina-contagem" v-model="contadores.horaFinal"/>

          <input type="submit" value="Consulta">
        </form>
      </div>
      <div class="tabela-contagem">
        <table>
          <thead>
          <tr>
            <th>Dia</th>
            <th>Hora Inicial</th>
            <th>Hora Final</th>
            <th>Refeitorio 1</th>
            <th>Refeitorio 2</th>
          </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ dia }}</td>
              <td>{{ horaInicial }}</td>
              <td>{{ horaFinal }}</td>
              <td>{{ refeitorio1 }}</td>
              <td>{{ refeitorio2 }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="relatorios">
      <h2>Gerar relatorio das catracas no Excel</h2>
      <div class="formulario-relatorio">
        <form @submit.prevent="gerarRelatorio()" @submit="loading = true">
          <label for="data_inicial_relatorio">Data Inicial</label>
          <input type="date" name="" id="data_inicial_relatorio" v-model="relatorio.dataInicial">
          <label for="data_final_relatorio">Data Final</label>
          <input type="date" name="" id="data_final_relatorio" v-model="relatorio.dataFinal">
          <label for="hora_inicial_relatorio">Hora Inicial</label>
          <input type="time" name="" id="hora_inicial_relatorio" v-model="relatorio.horaInicial">
          <label for="final_inicial_relatorio">Hora Final</label>
          <input type="time" name="" id="final_inicial_relatorio" v-model="relatorio.horaFinal">
          <input type="submit" value="Gerar">
        </form>
          <div class="carregando-animacao" v-show="loading">
            <div class="hollow-dots-spinner" :style="spinnerStyle">
              <div class="dot"></div>
              <div class="dot"></div>
              <div class="dot"></div>
              <div class="texto-carregando"><p><b>Carregando...aguarde</b></p></div>
            </div>
          </div>
      </div>
    </div>
  </div>
</template>

<script>

import Contagem from './domain/contagem/Contagem'
import Relatorio from './domain/relatorio/Relatorios'

export default {
  data() {
    return {
      relatorio: new Relatorio,
      contadores: new Contagem,
      refeitorio1: '',
      refeitorio2: '',
      horaInicial: '',
      horaFinal: '',
      dia: '',
      loading: false

    }
  },

  methods: {
    gerarRelatorio() {

      console.log(this.relatorio)

      this.$http.post('http://localhost:3000/relatorio/excel', this.relatorio, {responseType: 'arraybuffer'})
        .then((response) => {
          let blob = new Blob([response.data], {type: response.headers.get('content-type')});
          let link = document.createElement('a');
          link.href = window.URL.createObjectURL(blob);
          link.download = this.name;
          link.click();
          this.loading = false
        })
    },

    receberContadores(){
      this.$http.post('http://localhost:3000/contagem/refeitorios', this.contadores)
      .then(res => res.json())
      .then(result => {this.refeitorio1= result.refeitorio1, this.refeitorio2= result.refeitorio2,
        this.horaInicial = result.horaInicial, this.horaFinal = result.horaFinal, this.dia = result.dia}, err => console.log(err))
      }

    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

}

.tabela-contagem {
  margin: 0 auto;
  text-align: center;
  background: red;
  width: 100%;
  align: center;
}

.relatorios{


}
.texto-carregando{
  color:#00102f;
  font-size:25px;
  margin-left: -40px;
}
.carregando-animacao{
  width: 260px;
  position:relative;
  margin-left: -130px;
  left: 50%;
}


.hollow-dots-spinner, .hollow-dots-spinner * {
  box-sizing: border-box;
}

.hollow-dots-spinner {
  height: 25px;
  width: calc(80px * 3);
}

.hollow-dots-spinner .dot {
  width: 40px;
  height: 40px;
  margin: 0 calc(30px / 2);
  border: calc(20px / 5) solid #00102f;
  border-radius: 50%;
  float: left;
  transform: scale(0);
  animation: hollow-dots-spinner-animation 1000ms ease infinite 0ms;
}

.hollow-dots-spinner .dot:nth-child(1) {
  animation-delay: calc(300ms * 1);
}

.hollow-dots-spinner .dot:nth-child(2) {
  animation-delay: calc(300ms * 2);
}

.hollow-dots-spinner .dot:nth-child(3) {
  animation-delay: calc(300ms * 3);

}

@keyframes hollow-dots-spinner-animation {
  50% {
    transform: scale(1);
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}
</style>
