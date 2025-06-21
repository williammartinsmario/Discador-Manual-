-<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Discador William</title>
  <style>
    body { font-family: Arial; background: #f7f7f7; margin: 20px; }
    h1 { color: #333; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    th { background: #333; color: white; }
    button { padding: 6px 10px; margin: 2px; cursor: pointer; border: none; border-radius: 5px; }
    .ligar { background: #4CAF50; color: white; }
    .caixa { background: #f44336; color: white; }
    .atendeu { background: #2196F3; color: white; }
    .historico { background: orange; color: white; padding: 10px; margin-top: 10px;}
    .verde { background-color: #4CAF50; color: white; border-radius: 50%; padding: 5px; }
    .vermelha { background-color: #f44336; color: white; border-radius: 50%; padding: 5px; }
  </style>
</head>
<body>

<h1>Discador - William</h1>

<input type="file" id="fileInput" accept=".csv, .xlsx, .xls">
<button class="historico" onclick="exportarHistorico()">📄 Gerar Histórico</button>

<table id="tabela">
  <thead>
    <tr>
      <th>Nome</th>
      <th>Telefone</th>
      <th>Ações</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<script>
let historico = [];

document.getElementById('fileInput').addEventListener('change', function(e) {
    const file = e.target.files[0];
    const reader = new FileReader();
    reader.onload = function(e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, {type: 'array'});
        const sheet = workbook.Sheets[workbook.SheetNames[0]];
        const json = XLSX.utils.sheet_to_json(sheet, {header:1});
        preencherTabela(json);
    };
    reader.readAsArrayBuffer(file);
});

function formatarTelefone(tel) {
    tel = tel.toString().replace(/\D/g, '');
    if (tel.startsWith('19')) {
        return tel;
    } else if (tel.length >= 10) {
        return '0' + tel;
    }
    return tel;
}

function preencherTabela(data) {
    const tbody = document.querySelector('#tabela tbody');
    tbody.innerHTML = '';
    data.slice(1).forEach(linha => {
        const nome = linha[0];
        const telefoneOriginal = linha[1];
        if (!nome || !telefoneOriginal) return;
        const telefone = formatarTelefone(telefoneOriginal);
        const tr = document.createElement('tr');
        tr.innerHTML = `
            <td>${nome}</td>
            <td><a href="tel:${telefone}">${telefone}</a></td>
            <td>
                <button class="ligar" onclick="registrar('${nome}', '${telefone}', 'Ligou', this)" ondblclick="resetar(this)">📞 Ligar</button>
                <button class="caixa" onclick="registrar('${nome}', '${telefone}', 'Caixa Postal')">🔴 Caixa</button>
                <button class="atendeu" onclick="registrar('${nome}', '${telefone}', 'Atendeu')">💚 Atendeu</button>
            </td>
            <td id="status-${telefone}">🔘</td>
        `;
        tbody.appendChild(tr);
    });
}

function registrar(nome, telefone, status, btn) {
    document.getElementById('status-' + telefone).innerHTML =
        status === 'Atendeu' ? '💚' :
        status === 'Caixa Postal' ? '🔴' : '🔘';

    historico.push({
        Nome: nome,
        Telefone: telefone,
        Status: status,
        Data: new Date().toLocaleString()
    });

    if (status === 'Ligou' && btn) {
        btn.style.backgroundColor = '#f44336';
        btn.style.color = 'white';
    }
}

function resetar(btn) {
    btn.style.backgroundColor = '';
    btn.style.color = '';
}

function exportarHistorico() {
    if (historico.length === 0) {
        alert('Nenhum histórico registrado.');
        return;
    }
    const wb = XLSX.utils.book_new();
    const ws = XLSX.utils.json_to_sheet(historico);
    XLSX.utils.book_append_sheet(wb, ws, "Historico");
    XLSX.writeFile(wb, "historico_ligacoes.xlsx");
}
</script>

</body>
</html>
