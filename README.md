# controle-vendas
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Vendas v2.0</title>
    <script src="https://cdnjs.cloudflare.com"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f0f2f5; margin: 0; padding: 20px; }
        .container { max-width: 900px; margin: auto; background: white; padding: 25px; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        h2 { color: #1a73e8; border-bottom: 2px solid #e8f0fe; padding-bottom: 10px; margin-top: 30px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; font-weight: bold; margin-bottom: 5px; color: #444; }
        input, select { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        
        button { padding: 12px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; transition: 0.2s; }
        .btn-save { background-color: #00a884; color: white; width: 100%; font-size: 16px; margin-top: 10px; }
        .btn-save:hover { background-color: #008f70; }
        
        table { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
        th, td { border: 1px solid #eee; padding: 12px; text-align: left; }
        th { background-color: #f8f9fa; color: #555; }
        
        .actions { display: flex; gap: 5px; }
        .btn-pdf { background-color: #d93025; color: white; }
        .btn-wpp { background-color: #25d366; color: white; }
        
        /* Layout do Recibo PDF */
        #recibo-template { display: none; padding: 50px; font-family: Arial, sans-serif; }
        .recibo-header { text-align: center; border-bottom: 2px solid #000; margin-bottom: 20px; }
    </style>
</head>
<body>

<div class="container">
    <h2 style="margin-top:0">Registrar Nova Venda</h2>
    <form id="vendaForm">
        <div class="grid">
            <div class="form-group">
                <label>Nº Recibo</label>
                <input type="text" id="reciboNum" readonly style="background: #eee;">
            </div>
            <div class="form-group">
                <label>Data/Hora</label>
                <input type="text" id="dataHora" readonly style="background: #eee;">
            </div>
        </div>

        <div class="grid">
            <div class="form-group">
                <label>Mercadoria</label>
                <input type="text" id="mercadoria" required placeholder="Ex: Notebook">
            </div>
            <div class="form-group">
                <label>Comprador</label>
                <input type="text" id="comprador" required placeholder="Nome do cliente">
            </div>
        </div>

        <div class="grid">
            <div class="form-group">
                <label>Valor (R$)</label>
                <input type="number" step="0.01" id="valor" required placeholder="0,00">
            </div>
            <div class="form-group">
                <label>Forma de Pagamento</label>
                <select id="formaPagamento" required>
                    <option value="">Selecione...</option>
                    <option value="Pix">Pix</option>
                    <option value="Dinheiro">Dinheiro</option>
                    <option value="Cartão de Crédito">Cartão de Crédito</option>
                    <option value="Cartão de Débito">Cartão de Débito</option>
                    <option value="Transferência">Transferência Bancária</option>
                </select>
            </div>
        </div>

        <div class="form-group">
            <label>Anexar Comprovante (Opcional)</label>
            <input type="file" id="comprovante" accept="image/*">
        </div>

        <button type="submit" class="btn-save">FINALIZAR VENDA</button>
    </form>

    <h2>Histórico de Operações</h2>
    <div style="overflow-x:auto;">
        <table>
            <thead>
                <tr>
                    <th>Recibo</th>
                    <th>Cliente</th>
                    <th>Valor</th>
                    <th>Pagamento</th>
                    <th>Ações</th>
                </tr>
            </thead>
            <tbody id="corpoTabela"></tbody>
        </table>
    </div>
</div>

<!-- Estrutura do PDF -->
<div id="recibo-template">
    <div class="recibo-header">
        <h1>RECIBO DE PAGAMENTO</h1>
        <p>Controle de Vendas Profissional</p>
    </div>
    <div style="line-height: 1.8;">
        <p><strong>Número do Recibo:</strong> <span id="pdf-num"></span></p>
        <p><strong>Data de Emissão:</strong> <span id="pdf-data"></span></p>
        <hr>
        <p>Recebemos de <strong><span id="pdf-cliente"></span></strong>,</p>
        <p>a importância de <strong>R$ <span id="pdf-valor"></span></strong></p>
        <p>referente à compra de: <strong><span id="pdf-item"></span></strong>.</p>
        <p><strong>Forma de Pagamento:</strong> <span id="pdf-forma"></span></p>
    </div>
    <div style="margin-top: 100px; text-align: center;">
        <p>_________________________________________________</p>
        <p>Assinatura do Emitente</p>
    </div>
</div>

<script>
    let vendas = JSON.parse(localStorage.getItem('vendas_v2')) || [];

    function gerarId() {
        const agora = new Date();
        document.getElementById('reciboNum').value = 'REC-' + agora.getTime().toString().slice(-6);
        document.getElementById('dataHora').value = agora.toLocaleString('pt-BR');
    }

    document.getElementById('vendaForm').addEventListener('submit', function(e) {
        e.preventDefault();

        const venda = {
            id: document.getElementById('reciboNum').value,
            data: document.getElementById('dataHora').value,
            mercadoria: document.getElementById('mercadoria').value,
            comprador: document.getElementById('comprador').value,
            valor: document.getElementById('valor').value,
            forma: document.getElementById('formaPagamento').value
        };

        vendas.push(venda);
        localStorage.setItem('vendas_v2', JSON.stringify(vendas));
        
        this.reset();
        gerarId();
        renderizarTabela();
        alert('Venda salva com sucesso!');
    });

    function renderizarTabela() {
        const lista = document.getElementById('corpoTabela');
        lista.innerHTML = '';

        vendas.forEach((v, i) => {
            const tr = document.createElement('tr');
            tr.innerHTML = `
                <td>${v.id}</td>
                <td>${v.comprador}</td>
                <td>R$ ${parseFloat(v.valor).toFixed(2)}</td>
                <td><span style="background:#e8f0fe; padding:4px 8px; border-radius:4px; font-size:12px">${v.forma}</span></td>
                <td class="actions">
                    <button class="btn-pdf" onclick="baixarPDF(${i})">PDF</button>
                    <button class="btn-wpp" onclick="zap(${i})">Zap</button>
                </td>
            `;
            lista.appendChild(tr);
        });
    }

    function baixarPDF(i) {
        const v = vendas[i];
        document.getElementById('pdf-num').innerText = v.id;
        document.getElementById('pdf-data').innerText = v.data;
        document.getElementById('pdf-cliente').innerText = v.comprador;
        document.getElementById('pdf-item').innerText = v.mercadoria;
        document.getElementById('pdf-valor').innerText = parseFloat(v.valor).toFixed(2);
        document.getElementById('pdf-forma').innerText = v.forma;

        const containerPDF = document.getElementById('recibo-template');
        containerPDF.style.display = 'block';

        const config = {
            margin: 1,
            filename: `Recibo_${v.id}.pdf`,
            html2canvas: { scale: 2 },
            jsPDF: { unit: 'in', format: 'a4', orientation: 'portrait' }
        };

        html2pdf().set(config).from(containerPDF).save().then(() => {
            containerPDF.style.display = 'none';
        });
    }

    function zap(i) {
        const v = vendas[i];
        const msg = `*RECIBO DE VENDA - Nº ${v.id}*%0A%0A` +
                    `*Cliente:* ${v.comprador}%0A` +
                    `*Produto:* ${v.mercadoria}%0A` +
                    `*Valor:* R$ ${parseFloat(v.valor).toFixed(2)}%0A` +
                    `*Pagamento:* ${v.forma}%0A` +
                    `*Data:* ${v.data}%0A%0A` +
                    `Obrigado pela compra!`;
        
        window.open(`https://api.whatsapp.com{msg}`, '_blank');
    }

    gerarId();
    renderizarTabela();
</script>

</body>
</html>
