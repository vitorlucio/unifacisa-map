<template>
  <div class="container">
    <h2 class="text-center my-4">Localizador de Assento</h2>
    <div class="row justify-content-center">
      <div class="col-md-6">
        <div class="form-group">
          <label for="row">Fileira:</label>
          <input type="text" class="form-control" v-model="row" placeholder="Ex: M" />
        </div>
        <div class="form-group mt-3">
          <label for="seat">Assento:</label>
          <input type="number" class="form-control" v-model="seat" placeholder="Ex: 08" />
        </div>
        <button class="btn btn-primary mt-3 w-100" @click="findRoute">Traçar Rota</button>
        <button class="btn btn-secondary mt-3 w-100" @click="clearRoute">Limpar Rota</button>
      </div>
    </div>
    <div class="row justify-content-center mt-4">
      <div class="col-md-8">
        <canvas ref="canvas" width="800" height="800"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import config from "../../config.json";

export default {
  data() {
    return {
      row: "", // Linha do assento
      seat: null, // Número do assento
      graph: {}, // Grafo de conexões entre os assentos
      img: null, // Imagem da planta baixa
      positions: {}, // Posições dos assentos no canvas
      seatSet: null, // Conjunto de assentos (M33 ou G33)
      routeDrawn: false // Controle do estado da rota (se foi traçada ou não)
    };
  },
  methods: {
    initializeSeatData(seatSetKey) {
      const seatSet = config.seatSets[seatSetKey];
      if (seatSet) {
        this.positions = seatSet.positions;
        this.graph = seatSet.graph;
      } else {
        alert("Conjunto de assentos não encontrado.");
      }
    },

    selectSeatSet() {
      if (this.row && this.seat) {
        const seatSetKey = `${this.row}${this.seat}`;
        if (config.seatSets[seatSetKey]) {
          this.seatSet = seatSetKey;
          this.initializeSeatData(seatSetKey);
        } else {
          alert("Conjunto de assentos inválido para a fileira e assento especificados.");
        }
      }
    },

    findShortestPath(start, end) {
      const distances = {};
      const previous = {};
      const nodes = new Set(Object.keys(this.graph));

      nodes.forEach(node => (distances[node] = Infinity));
      distances[start] = 0;

      while (nodes.size) {
        const closestNode = Array.from(nodes).reduce((minNode, node) =>
          distances[node] < distances[minNode] ? node : minNode
        );

        if (closestNode === end) {
          let path = [];
          let temp = closestNode;
          while (previous[temp]) {
            path.push(temp);
            temp = previous[temp];
          }
          return path.concat(start).reverse();
        }

        nodes.delete(closestNode);

        this.graph[closestNode].forEach(neighbor => {
          const alt = distances[closestNode] + 1;
          if (alt < distances[neighbor]) {
            distances[neighbor] = alt;
            previous[neighbor] = closestNode;
          }
        });
      }
      return [];
    },

    findRoute() {
      if (!this.img) {
        alert("A imagem não está carregada ainda.");
        return;
      }
      this.selectSeatSet();

      const start = "A1"; // Entrada fixa
      const end = `${this.row}${this.seat}`;

      const path = this.findShortestPath(start, end);
      if (path.length) {
        this.routeDrawn = true; // Marca que a rota foi traçada
        this.drawRoute(path);
      } else {
        alert("Caminho não encontrado.");
      }
    },

    drawRoute(path) {
      if (!this.img) {
        console.error("Imagem não carregada, não é possível desenhar a rota.");
        return;
      }

      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");

      // Sombreia o fundo em cinza se a rota foi traçada
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (this.routeDrawn) {
        ctx.filter = "grayscale(100%)"; // Aplica filtro cinza
      } else {
        ctx.filter = "none"; // Sem filtro se não houver rota
      }

      ctx.drawImage(this.img, 0, 0, canvas.width, canvas.height);
      ctx.filter = "none"; // Removendo o filtro para desenhar a rota

      // Desenha a rota
      ctx.beginPath();
      ctx.strokeStyle = "red";
      ctx.lineWidth = 3;

      path.forEach((node, index) => {
        const [x, y] = this.positions[node];
        if (index === 0) {
          ctx.moveTo(x, y);
        } else {
          ctx.lineTo(x, y);
        }
      });
      ctx.stroke();

      // Destaca o início com um quadrado vermelho
      const [startX, startY] = this.positions[path[0]];
      ctx.fillStyle = "red";
      ctx.fillRect(startX - 5, startY - 5, 10, 10); // Quadrado de 10x10 pixels
      
    },

    clearRoute() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(this.img, 0, 0, canvas.width, canvas.height);

      this.row = "";
      this.seat = null;
      this.routeDrawn = false; // Reseta o estado da rota
      this.drawRoute([]); // Reinicia a exibição da seta
    },

    loadImage() {
      this.img = new Image();
      this.img.src = require("@/assets/mapa-teatro-facisa.png");

      this.img.onload = () => {
        const canvas = this.$refs.canvas;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(this.img, 0, 0, canvas.width, canvas.height); // Imagem inicial colorida
      };

      this.img.onerror = () => {
        console.error("Erro ao carregar a imagem:", this.img.src);
        alert("A imagem não pôde ser carregada. Verifique o caminho.");
      };
    }
  },
  mounted() {
    this.loadImage();
  }
};
</script>

<style scoped>
canvas {
  border: 1px solid black;
}
</style>
