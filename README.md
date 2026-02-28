<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Vendas Mobile</title>
    <script src="https://cdnjs.cloudflare.com"></script>
    <style>
        :root { --primary: #075e54; --secondary: #25d366; --bg: #e5ddd5; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background: var(--bg); margin: 0; padding: 10px; color: #333; }
        
        .app-card { background: white; border-radius: 15px; padding: 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; }
        h2 { font-size: 1.2rem; color: var(--primary); margin-top: 0; border-bottom: 2px solid #eee; padding-bottom: 10px; }
        
        .form-group { margin-bottom: 15px; }
        label { display: block; font-size: 0.85rem; font-weight: 600; margin-bottom: 5px; color: #666; }
        input, select { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 8px; font-size: 16px; box-sizing: border-box; background: #fafafa; }
        
        .row { display: flex; gap: 10px; }
        .row > div { flex: 1; }

        button { width: 100%; padding: 15px; border: none; border-radius: 10px; font-weight: bold; font-size: 1rem; cursor: pointer; transition: 0.2s; display: flex; align-items: center; justify-content: center; gap: 8px; }
        .btn-main { background: var(--primary); color: white; margin-top: 10px; }
        .btn-pdf { background: #e74c3c; color: white; margin-bottom: 5px; }
        .btn-wpp { background: var(--secondary); color: white; }

        /* Lista de Vendas estilo Mobile */
        .venda-item { background: white; border-radius: 12px; padding: 15px; margin-bottom: 10px; border-left: 5px solid var(--primary); box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .venda-header { display: flex; justify-content: space-between; font-weight: bold; margin-bottom: 8px; font-size: 0.9rem; }
        .venda-detalhes { font-size: 0.9rem; color: #555; line-height: 1.5; margin-bottom: 10px; }
        .venda-acoes { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

        /* Estilo do PDF (Escondido) */
        #recibo-print { display: none; width: 300px; padding: 20px; font-family: 'Courier New', Courier, monospace; }
    </style>
</head>
<body>

<div class="app-card">
    <h2>📝 Nova Venda</h2>
    <form id="vendaForm">
        <div class="row">
            <div class="form-group">
                <label>Nº Recibo</label>
                <input type="text" id="reciboNum" readonly style="color: #999;">
            </div>
            <div class="form-group">
                <label>Data</label>
                <input type="text" id="dataHora" readonly style="color: #999;">
            </div>
        </div>

        <div class="form-group">
            <label>Mercadoria</label>
            <input type="text" id="mercadoria" required placeholder="Nome do produto">
        </div>

        <div class="row">
            <div class="form-group">
                <label>Quantidade</label>
                <input type="number" id="qtd" value="1" min="1" required>
            </div>
            <div class="form-group">
                <label>Valor Unit. (R$)</label>
                <input type="number" step="0.01" id="valor" required placeholder="0,00">
            </div>
        </div>

        <div class="form-group">
            <label>Nome do Comprador</label>
            <input type="text" id="comprador" required placeholder="Quem comprou?">
        </div>

        <div class="form-group">
            <label>Forma de Pagamento</label>
            <select id="formaPagamento" required>
                <option value="Pix">Pix</option>
                <option value="Dinheiro">Dinheiro</option>
                <option value="Cartão">Cartão</option>
            </select>
        </div>

        <div class="form-group">
            <label>Comprovante (Foto)</label>
            <input type="file" id="comprovante" accept="image/*">
        </div>

        <button type="submit" class="btn-main">✅ SALVAR OPERAÇÃO</button>
    </form>
</div>

<h2 style="padding-left: 10px;">📋 Histórico Recente</h2>
<div id="listaVendas"></div>

<!-- Template para Impressão -->
<div id="recibo-print">
    <center>
        <h3>RECIBO DE VENDA</h3>
        <p>--------------------------</p>
    </center>
    <p><b>RECIBO:</b> <span id="p-id"></span></p>
    <p><b>DATA:</b> <span id="p-data"></span></p>
    <p><b>CLIENTE:</b> <span id="p-cliente"></span></p>
    <p>--------------------------</p>
    <p><b>PROD:</b> <span id="p-prod"></span></p>
    <p><b>QTD:</b> <span id="p-qtd"></span></p>
    <p><b>TOTAL:</b> R$ <span id="p-total"></span></p>
    <p><b>PAGTO:</b> <span id="p-forma"></span></p>
    <p>--------------------------</p>
    <center><p>Obrigado pela preferência!</p></center>
</div>

<script>
    let vendas = JSON.parse(localStorage.getItem('vendas_mobile')) || [];

    function init() {
        const agora = new Date();
        document.getElementById('reciboNum').value = Math.floor(Date.now() / 1000).toString().slice(-6);
        document.getElementById('dataHora').value = agora.toLocaleDateString() + ' ' + agora.toLocaleTimeString([], {hour: '2-min', minute:'2-min'});
    }

    document.getElementById('vendaForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const v = {
            id: document.getElementById('reciboNum').value,
            data: document.getElementById('dataHora').value,
            prod: document.getElementById('mercadoria').value,
            qtd: document.getElementById('qtd').value,
            valor: document.getElementById('valor').value,
            cliente: document.getElementById('comprador').value,
            forma: document.getElementById('formaPagamento').value,
            total: (document.getElementById('qtd').value * document.getElementById('valor').value).toFixed(2)
        };
        vendas.unshift(v);
        localStorage.setItem('vendas_mobile', JSON.stringify(vendas));
        this.reset();
        init();
        render();
        window.scrollTo(0, document.body.scrollHeight);
    });

    function render() {
        const container = document.getElementById('listaVendas');
        container.innerHTML = vendas.map((v, i) => `
            <div class="venda-item">
                <div class="venda-header">
                    <span>#${v.id}</span>
                    <span style="color:var(--primary)">R$ ${v.total}</span>
                </div>
                <div class="venda-detalhes">
                    <b>${v.prod}</b> (${v.qtd} un)<br>
                    Cliente: ${v.cliente}<br>
                    Pagto: ${v.forma} - ${v.data}
                </div>
                <div class="venda-acoes">
                    <button class="btn-pdf" onclick="gerarPDF(${i})">📄 PDF</button>
                    <button class="btn-wpp" onclick="zap(${i})">📱 Zap</button>
                </div>
            </div>
        `).join('');
    }

    function gerarPDF(i) {
        const v = vendas[i];
        document.getElementById('p-id').innerText = v.id;
        document.getElementById('p-data').innerText = v.data;
        document.getElementById('p-cliente').innerText = v.cliente;
        document.getElementById('p-prod').innerText = v.prod;
        document.getElementById('p-qtd').innerText = v.qtd;
        document.getElementById('p-total').innerText = v.total;
        document.getElementById('p-forma').innerText = v.forma;

        const el = document.getElementById('recibo-print');
        el.style.display = 'block';
        html2pdf().from(el).set({
            margin: 10,
            filename: 'recibo-'+v.id+'.pdf',
            jsPDF: { unit: 'mm', format: 'a6' }
        }).save().then(() => el.style.display = 'none');
    }

    function zap(i) {
        const v = vendas[i];
        const texto = encodeURIComponent(
            `*RESUMO DA COMPRA - RECIBO #${v.id}*\n\n` +
            `*Produto:* ${v.prod}\n` +
            `*Qtd:* ${v.qtd}\n` +
            `*Total:* R$ ${v.total}\n` +
            `*Pagamento:* ${v.forma}\n` +
            `*Cliente:* ${v.cliente}\n` +
            `*Data:* ${v.data}\n\n` +
            `_Comprovante emitido via Sistema Mobile_`
        );
        window.open(`https://api.whatsapp.com{texto}`);
    }

    init();
    render();
</script>

</body>
</html>
