# Engenharia de Dados

Este é o repositório do curso de Engenharia de Dados do Insper, onde você encontrará todo o material necessário para acompanhar as aulas e realizar as atividades práticas.

## Acesso ao Material

### GitHub Pages

Para acessar o material, utilize o link do [GitHub Pages](https://insper.github.io/dataeng/).

### Executar localmente

Caso queira executar o material localmente, siga os passos abaixo:
1. **Clone o repositório**:
   ```bash
   git clone https://github.com/insper/dataeng.git
   ```

2. **Navegue até o diretório do projeto**:
   ```bash
   cd dataeng
   ```

3. **Instale as dependências**: recomendo que crie um ambiente virtual para evitar conflitos com outras bibliotecas Python que você possa ter instalado. Você pode usar `venv` ou `conda` para isso. Aqui está um exemplo usando `venv`:
   ```bash
   python -m venv venv
   source venv/bin/activate  # No Windows use: venv\Scripts\activate
   pip install -r requirements.txt
   ```

4. **Execute o servidor local**:
   ```bash
   mkdocs serve
   ```

Acesse o material no seu navegador em `http://localhost:8000`.

# Referências
- [MkDocs Cover Page](https://github.com/tylerdotrar/mkdocs-coverpage)