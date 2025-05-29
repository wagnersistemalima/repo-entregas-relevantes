# repo-entregas-relevantes

# Pix Shield

## Descrição do Projeto

O **Pix Shield** é uma solução desenvolvida para monitorar as dependências críticas do sistema PIX, garantindo informações atualizadas sobre a disponibilidade de instituições financeiras. O objetivo principal é melhorar a confiabilidade e a experiência do usuário, expondo os dados de forma estruturada para consumo no frontend. A solução foi projetada para ser expansível e de fácil integração com futuras dependências.

### Principais Funcionalidades

- Monitoramento periódico das dependências críticas do sistema PIX.
- Coleta de informações de disponibilidade de instituições financeiras.
- Armazenamento em cache para respostas rápidas e consistentes.
- Exposição de dados estruturados via API para consumo no frontend.
- Integração com BFFs para adaptação e entrega de dados ao frontend.

---

## Perguntas e Respostas Técnicas

### **Arquitetura e Design**

    A solução surgiu para diminuir 1% dos erros no fluxo de transferencia.

		E a indisponibilidade das instituição representaria 0,20%


**Pergunta:** Fale sobre a inversao de controle e injeção de dependencia?
**Resposta:** Inversão de Controller é um princípio de design onde o framework assume o papel de criar e gerenciar os objetos e suas dependências, em vez de o próprio código da aplicação fazer isso diretamente. Isso promove um design mais desacoplado e facilita a manutenção e testes. A injeção de dependência (DI) é uma técnica específica para implementar IoC, onde as dependências são fornecidas a uma classe em vez de serem criadas internamente. Isso facilita a substituição de implementações e melhora a modularidade do código.

**Pergunta:** Como você definiu a arquitetura do Pix Shield para garantir escalabilidade e fácil integração com futuras dependências?  
**Resposta:** A arquitetura foi baseada em microsserviços, com separação clara de responsabilidades. Utilizamos Redis como cache devido à sua baixa latência e alta performance, garantindo respostas rápidas em tempo real.

**Pergunta:** Por que a escolha de microsserviços foi mais adequada para o Pix Shield em comparação com uma arquitetura monolítica?
**Resposta:** A escolha por microsserviços foi motivada pela necessidade de escalabilidade, flexibilidade e independência no desenvolvimento e implantação de cada componente. Essa abordagem permite que novas dependências sejam adicionadas sem impactar os serviços existentes, além de facilitar a manutenção e a evolução do sistema.

**Pergunta:** Quais foram os principais desafios ao modificar os contratos das APIs do PIX e como você os superou?  
**Resposta:** O principal desafio foi adicionar as informações de disponibilidade do sistema financeiro ao contrato existente sem impactar os consumidores atuais. Para isso, anexamos ao payload de resposta os dados de disponibilidade, mantendo o contrato compatível com as implementações existentes. O frontend foi responsável por interpretar o status e exibir um banner na tela de revisão do Pix, alertando o usuário caso o sistema financeiro de destino estivesse indisponível. Essa abordagem garantiu uma integração suave e evitou a necessidade de versionamento das APIs.

**Pergunta:** Como você garantiu a compatibilidade entre os contratos das APIs ao adicionar novas informações?
**Resposta:** Garantimos a compatibilidade dos contratos das APIs ao adicionar novas informações anexando os dados de forma opcional no payload de resposta. Esse anexo será ignorado por versões antigas do frontend e somente interpretado pelas versões atualizadas do iOS e Android, garantindo que os consumidores existentes continuem funcionando sem alterações.


**Pergunta:** Quais foram os critérios para escolher Redis em vez de DynamoDB, além da simplicidade de configuração?
**Resposta:** Além da simplicidade de configuração, escolhemos o Redis por sua capacidade de fornecer operações de leitura e escrita em tempo real com baixa latência.

**Pergunta:** Por que o Redis foi escolhido como solução de cache para o Pix Shield? Você considerou outras alternativas?  
**Resposta:** O Redis foi escolhido como solução de cache devido à sua alta performance, baixa latência e suporte a estruturas de dados avançadas, essenciais para o Pix Shield. Consideramos o DynamoDB como alternativa, mas optamos pelo Redis por sua simplicidade de configuração e eficiência em operações de leitura e escrita em tempo real, que são cruciais para o monitoramento de disponibilidade do sistema financeiro.

---

### **Desenvolvimento e Implementação**

**Pergunta:** Como foi implementada a aplicação de monitoramento que executa a cada 30 segundos? Quais cuidados foram tomados para evitar sobrecarga no sistema?  
**Resposta:** A aplicação foi implementada como um job agendado utilizando `@Scheduled` do Spring Boot. Configuramos limites de timeout e retries, além de monitorar o consumo de recursos. 

A anotação @Scheduled no Spring é usada para executar tarefas agendadas de forma automática em intervalos de tempo definidos. Ela é aplicada em métodos e funciona em conjunto com o suporte a tarefas agendadas do Spring.

Configuração
Para usar o @Scheduled, é necessário habilitar o agendamento de tarefas na aplicação com a anotação @EnableScheduling em uma classe de configuração ou na classe principal.


**Pergunta:** Quais estratégias foram usadas para garantir a consistência e a confiabilidade dos dados expostos pelo Pix Shield?  
**Resposta:** Utilizamos a configuração do Redis para limpar o cache por meio de um TTL (Time to Live) curto, garantindo que os dados sejam atualizados regularmente. Além disso, implementamos um job agendado que é acionado a cada 30 segundos para realizar consultas ao sistema de monitoramento (Sonar) e atualizar a base do cache com as informações mais recentes.

**Pergunta:** Como você garantiu que o Pix Shield fosse facilmente expansível para futuras dependências?
**Resposta:** Para garantir a expansibilidade do Pix Shield, projetamos a arquitetura com uma abordagem modular, permitindo que novas dependências sejam adicionadas como novos serviços ou módulos. Utilizamos APIs bem definidas e contratos claros entre os serviços, facilitando a integração de novas fontes de dados sem impactar os componentes existentes.

**Pergunta:** Como você lidou com possíveis falhas no monitoramento das dependências críticas?  
**Resposta:** Quando há falha na comunicação entre o BFF e a API que consulta o Redis, devolvemos o status de disponibilidade financeira como INDETERMINADO. O frontend interpreta esse status como disponível, pois não podemos interromper o fluxo da transação. Registramos logs de erro no backend para essa condição e planejamos criar alertas na segunda fase do projeto.

**Pergunta:** Como você lidou com possíveis inconsistências entre os dados do Redis e os dados reais das instituições financeiras?
**Resposta:** Para mitigar inconsistências, configuramos um TTL (Time to Live) curto no Redis

**Pergunta:** Na parte Tecnica , o que traz segurança na implementação dessa solução Pix shield?
**Resposta:** Test-Driven Development (TDD): A prática de TDD foi aplicada para garantir que cada funcionalidade fosse desenvolvida com testes automatizados desde o início. Isso ajuda a identificar e corrigir vulnerabilidades ou comportamentos inesperados antes mesmo da implementação final

1 -> RED. Utilizando o conceito de escrever primeiro o teste que falha, pos a funcionalidade ainda nao foi implementada
2 -> GREEN: Implementa-se o codigo minimo para que o teste passe
3 -> REFACTOR: Refatora o codigo para melhorar sua qualidade

---

### **Integração e Infraestrutura**

**Pergunta:** Como foi o processo de provisionamento do gateway na AWS? Quais serviços da AWS foram utilizados?  
**Resposta:** Utilizamos o API Gateway para expor as APIs.

**Pergunta:** Quais foram os principais desafios ao integrar o Pix Shield com os BFFs e o frontend?  
**Resposta:** O principal desafio foi garantir contratos bem definidos e consistentes. Resolvemos isso com testes de integração automatizados.

---

### **Performance e Monitoramento**

**Pergunta:** Quais métricas foram definidas para monitorar a performance do Pix Shield?  
**Resposta:** Monitoramos tempo de resposta., taxa de sucesso das requisições, disponibilidade do sistema financeiro e consumo de recursos.

**Pergunta:** Como você validou que o tempo de resposta do sistema atendia aos requisitos de confiabilidade e experiência do usuário?  
**Resposta:** Na fase 1, validamos o tempo de resposta do sistema apenas no backend. Ainda não implantamos o frontend, mas os resultados obtidos indicam que o sistema atende aos requisitos de confiabilidade. Em fases posteriores, planejamos implementar alertas e dashboards para melhorar a observabilidade e monitorar o desempenho em tempo real.

**Pergunta:** Quais estratégias foram implementadas para lidar com picos de tráfego no sistema?  
**Resposta:** Configuramos escalabilidade automática no Redis e no gateway para suportar picos de tráfego.

---

### **Resultados e Aprendizados**

**Pergunta:** Quais foram os principais aprendizados técnicos ao desenvolver o Pix Shield?  
**Resposta:** Aprendemos a importância de definir contratos claros entre serviços, a necessidade de monitoramento contínuo e a eficácia do Redis como solução de cache para sistemas críticos.

**Pergunta:** Como você mediu o impacto da solução na experiência do usuário e na confiabilidade do sistema PIX?  
**Resposta:** Ainda não medimos o impacto diretamente na experiência do usuário, pois o frontend ainda não foi implantado. 

**Pergunta:** Se tivesse que refazer o projeto, o que faria de forma diferente para melhorar ainda mais o resultado?  
**Resposta:** Investiria mais tempo na automação de processos de monitoramento, das métricas de performance e na criação de dashboards para visualização em tempo real. Além disso, consideraria a implementação de testes de carga mais robustos para garantir a resiliência do sistema sob alta demanda.

---

## Tecnologias Utilizadas

- **Linguagem:** Java
- **Framework:** QuickCloud
- **Cache:** Redis
- **Infraestrutura:** AWS (API Gateway)
- **Ferramentas de Monitoramento:** DataDog, Grafana

---

## Como Contribuir

1. Faça um fork do repositório.
2. Crie uma branch para sua feature (`git checkout -b minha-feature`).
3. Faça commit das suas alterações (`git commit -m 'Minha nova feature'`).
4. Envie um push para a branch (`git push origin minha-feature`).
5. Abra um Pull Request.

Contribuições são bem-vindas!