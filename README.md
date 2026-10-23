# Conversor de Arquivos

Site de página única para converter vários arquivos de uma vez entre PDF, Word e Excel.
A conversão acontece inteiramente no navegador de quem usa: nenhum arquivo é enviado para o servidor.

## Conversões

| De | Para | Como funciona |
|---|---|---|
| PDF | Word (.docx) | Extrai texto, parágrafos, títulos, negrito, realces (marca-texto) e tabelas com bordas. |
| Word (.docx) | PDF | Renderiza o documento em páginas A4 (margens de 18 mm), sem cortar linhas na quebra de página. |
| PDF | Excel (.xlsx) | Tabelas com bordas viram células (com mesclagens); texto sem bordas é separado em colunas pelo alinhamento. Números como `1.234,56` viram números. |
| Excel (.xlsx, .xls, .csv, .ods) | PDF | Uma tabela por aba, com texto selecionável e numeração de páginas. Muitas colunas → página em paisagem. |

## Como usar

1. Escolha a conversão (PDF → Word, Word → PDF, PDF → Excel ou Excel → PDF).
2. Envie os arquivos: arraste arquivos ou pastas para a área tracejada, ou use **Selecionar arquivos** / **Selecionar pasta**. Arquivos de outro formato são ignorados.
3. Clique em **Converter** e baixe cada arquivo ou todos em **Baixar tudo (.zip)**. Arquivos vindos de pastas mantêm as subpastas dentro do .zip.

**Modo avançado:** permite misturar formatos na mesma fila. Word e Excel viram PDF; para cada PDF você escolhe se vira Word ou Excel.

**Opção do Excel:** ao converter PDF → Excel, dá para colocar cada página em uma aba ou tudo em uma aba só.

## Instalação

Não há build nem backend. Basta servir o `index.html` como arquivo estático.

### nginx

```nginx
server {
    listen 80;
    server_name conversor.seudominio.com;

    root /var/www/conversor;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo mkdir -p /var/www/conversor
sudo cp index.html /var/www/conversor/
sudo nginx -t && sudo systemctl reload nginx
```

Para HTTPS, use o Certbot: `sudo certbot --nginx -d conversor.seudominio.com`.

### Local

Abra o `index.html` direto no navegador, ou sirva a pasta:

```bash
python3 -m http.server 8080
```

## Dependências

Carregadas por CDN (é preciso internet no navegador de quem usa):

| Biblioteca | Versão | Uso |
|---|---|---|
| [pdf.js](https://mozilla.github.io/pdf.js/) | 3.11.174 | Ler texto e desenho dos PDFs |
| [mammoth](https://github.com/mwilliamson/mammoth.js) | 1.6.0 | Ler .docx |
| [docx](https://docx.js.org/) | 8.5.0 | Gerar .docx |
| [SheetJS (xlsx)](https://sheetjs.com/) | 0.18.5 | Ler e gerar planilhas |
| [jsPDF](https://github.com/parallax/jsPDF) + [AutoTable](https://github.com/simonbengtsson/jsPDF-AutoTable) | 2.5.1 / 3.8.2 | Gerar PDFs |
| [html2canvas](https://html2canvas.hertzen.com/) | 1.4.1 | Renderizar páginas do Word → PDF |
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | Gerar o .zip |

Para funcionar sem internet, baixe esses arquivos, coloque-os ao lado do `index.html` e troque os endereços nas tags `<script>`.

## Limitações

- **PDFs digitalizados** (só imagem) não têm texto para extrair; seria preciso OCR.
- **Imagens e logos** do PDF não são levados para o Word.
- **Tabelas sem linhas** desenhadas viram texto alinhado por tabulação no Word (no Excel, saem em colunas).
- **Word → PDF** gera páginas como imagem: o texto do PDF resultante não é selecionável, e fontes/layouts muito elaborados podem diferir do original.
- **Arquivos .doc** (Word 97–2003) não são suportados; salve como .docx antes.
- **PDFs com senha** precisam ter a proteção removida.
- Arquivos muito grandes dependem da memória do navegador; em lotes grandes, converta em partes.

## Estrutura do código

Tudo está em `index.html`:

- `analyzePage()` — lê texto, fontes em negrito, retângulos coloridos e linhas de cada página do PDF.
- `buildTables()` — agrupa linhas horizontais e verticais que se tocam e monta a grade das tabelas (com células mescladas).
- `pdfToDocx()`, `pdfToXlsx()`, `docxToPdf()`, `xlsxToPdf()` — as quatro conversões.
- `addFiles()`, `readEntry()` — entrada de arquivos e leitura recursiva de pastas arrastadas.
