# bacboo-analizer
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Analisador de BacBo</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background: #f2f2f2;
    }
    .container {
      max-width: 700px;
      margin: 0 auto;
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    input[type="number"] {
      width: 50px;
      margin: 5px;
    }
    .results {
      margin-top: 20px;
    }
    .sugestao {
      margin-top: 15px;
      padding: 10px;
      background-color: #e0f7fa;
      border-left: 5px solid #00796b;
    }
    #chartContainer {
      margin-top: 30px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Analisador de BacBo</h1>
    <p>Insira os valores dos dados (dois para cada jogador):</p>

    <div>
      <label>Jogador A: </label>
      <input type="number" id="a1" min="1" max="6">
      <input type="number" id="a2" min="1" max="6">
    </div>
    <div>
      <label>Jogador B: </label>
      <input type="number" id="b1" min="1" max="6">
      <input type="number" id="b2" min="1" max="6">
    </div>

    <button onclick="analisar()">Analisar</button>

    <div class="results" id="result"></div>

    <div id="chartContainer">
      <canvas id="victoryChart" width="400" height="400"></canvas>
    </div>
  </div>

  <script>
    const historico = [];
    let chart = null;

    function analisar() {
      const a1 = parseInt(document.getElementById('a1').value);
      const a2 = parseInt(document.getElementById('a2').value);
      const b1 = parseInt(document.getElementById('b1').value);
      const b2 = parseInt(document.getElementById('b2').value);

      if ([a1, a2, b1, b2].some(n => isNaN(n) || n < 1 || n > 6)) {
        alert("Por favor, insira valores entre 1 e 6 para todos os dados.");
        return;
      }

      const somaA = a1 + a2;
      const somaB = b1 + b2;
      const vencedor = somaA > somaB ? "Jogador A" : somaA < somaB ? "Jogador B" : "Empate";

      historico.push({ a1, a2, b1, b2, somaA, somaB, vencedor });

      const ultimos = historico.slice(-5).map((h, i) => Rodada ${historico.length - 5 + i + 1}: A(${h.somaA}) vs B(${h.somaB}) - ${h.vencedor}).join('<br>');
      const sugestao = gerarSugestao();

      document.getElementById("result").innerHTML = `
        <p>Soma A: <strong>${somaA}</strong> | Soma B: <strong>${somaB}</strong></p>
        <p><strong>Resultado:</strong> ${vencedor}</p>
        <hr>
        <h3>Últimos Resultados:</h3>
        ${ultimos}
        <div class="sugestao"><strong>Sugestão com base no histórico:</strong><br>${sugestao}</div>
      `;

      atualizarGrafico();
    }

    function gerarSugestao() {
      if (historico.length < 3) return "Insira pelo menos 3 rodadas para gerar uma sugestão.";

      const ultimos = historico.slice(-5);
      const contagem = { "Jogador A": 0, "Jogador B": 0, "Empate": 0 };
      ultimos.forEach(h => contagem[h.vencedor]++);

      let tendencia = Object.entries(contagem).sort((a, b) => b[1] - a[1])[0];

      if (tendencia[1] >= 3) {
        return Tendência observada: <strong>${tendencia[0]}</strong> venceu ${tendencia[1]} das últimas 5 rodadas. Pode ser vantajoso observar esse padrão.;
      }
      return "Nenhuma tendência forte identificada nas últimas rodadas.";
    }

    function atualizarGrafico() {
      const contagem = { "Jogador A": 0, "Jogador B": 0, "Empate": 0 };
      historico.forEach(h => contagem[h.vencedor]++);

      const data = {
        labels: ["Jogador A", "Jogador B", "Empate"],
        datasets: [{
          data: [contagem["Jogador A"], contagem["Jogador B"], contagem["Empate"]],
          backgroundColor: ["#42a5f5", "#66bb6a", "#ffca28"]
        }]
      };

      const config = {
        type: 'pie',
        data: data,
        options: {
          responsive: true,
          plugins: {
            legend: {
              position: 'bottom'
            }
          }
        }
      };

      if (chart) {
        chart.destroy();
      }
      const ctx = document.getElementById('victoryChart').getContext('2d');
      chart = new Chart(ctx, config);
    }
  </script>
</body>
</html>
