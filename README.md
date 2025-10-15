# TCC - Integração e Entrega Contínua (CI/CD)

Repositório desenvolvido para o Trabalho de Conclusão de Curso (TCC) na UTFPR, com foco em **Integração Contínua (CI)** e **Entrega Contínua (CD)**.  
O objetivo deste projeto é **comparar as funcionalidades e o desempenho das ferramentas Jenkins, GitLab CI/CD e GitHub Actions**, utilizando pipelines padronizadas em um projeto base.

---

## 🎯 Objetivos

- Identificar as principais características e limitações de cada ferramenta de CI/CD.
- Avaliar facilidade de configuração, tempo de execução e suporte a integrações.
- Oferecer recomendações práticas para a escolha da ferramenta mais adequada em diferentes contextos de desenvolvimento.

---

## 🛠️ Tecnologias utilizadas

- **.NET** – Projeto base e execução de testes automatizados com **xUnit**.
- **Docker** – Criação de imagens containerizadas.
- **Docker Hub** – Publicação das imagens.
- **Trivy** – Escaneamento de vulnerabilidades em imagens Docker.
- **YAML** – Definição de pipelines no GitHub Actions e GitLab CI/CD.
- **Groovy (Jenkinsfile)** – Definição de pipelines no Jenkins.

---

## 📂 Estrutura do repositório

```
├── src/                  # Código-fonte do projeto base (.NET)
├── tests/                # Testes automatizados com xUnit
├── .github/workflows/    # Pipelines do GitHub Actions
├── .gitlab-ci.yml        # Pipeline do GitLab CI/CD
├── Jenkinsfile           # Pipeline declarativa para Jenkins
├── Dockerfile            # Build da aplicação em container
└── README.md             # Este arquivo
```

## Como executar as pipelines

### GitHub Actions

- As pipelines são executadas automaticamente no GitHub em cada `push` ou `pull request`, conforme definido em `.github/workflows/`.

### GitLab CI/CD

- A pipeline é executada automaticamente no GitLab a cada `push`, conforme definido no `.gitlab-ci.yml`.

### Jenkins

1. Subir um container Jenkins com 2 vCPUs e 4 GB RAM.
2. Configurar os plugins necessários e criar um pipeline apontando para o `Jenkinsfile`.

---

## 📜 Licença

Este repositório é disponibilizado para fins acadêmicos e de pesquisa.  
Sinta-se à vontade para consultar e adaptar o conteúdo, citando a fonte.

---

## 👨‍💻 Autor

**Murilo Nunes Marçal**  
Trabalho de Conclusão de Curso – UTFPR, 2025
