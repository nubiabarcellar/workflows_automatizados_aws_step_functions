# Workflows Automatizados com AWS Step Functions

## Descrição Geral

Este repositório documenta a prática e os aprendizados adquiridos no desafio “Workflows Automatizados com AWS Step Functions”.
O objetivo principal foi projetar, implementar e documentar um workflow serverless automatizado integrando diferentes serviços da AWS — principalmente Step Functions, Lambda, SNS e SQS — para demonstrar como é possível orquestrar processos complexos de forma gerenciável e escalável.

## AWS Step Functions

O AWS Step Functions é um serviço de orquestração de workflows que permite coordenar múltiplos serviços AWS em um fluxo visual.
A lógica do workflow é definida em formato JSON, com estados, transições e condições que descrevem como cada etapa será executada.

Estados representam as etapas do processo (por exemplo, executar uma Lambda, enviar uma mensagem SNS, aguardar uma resposta, etc.).

Transições definem o próximo passo dependendo do resultado de cada estado.

Benefícios principais:

Integração nativa com mais de 200 serviços AWS;

Resiliência, controle de erros e reexecução automática;

Monitoramento visual e histórico de execução no console.

## AWS Lambda

O AWS Lambda é um serviço de computação serverless que permite executar código sob demanda sem provisionar servidores.
É frequentemente usado dentro de Step Functions para processar dados, aplicar regras de negócio e acionar outros serviços.

Exemplo de função Lambda:

import json
import boto3

def lambda_handler(event, context):
    print("Evento recebido:", event)
    message = f"Processamento realizado com sucesso para o item: {event.get('item', 'desconhecido')}"
    return {"status": "SUCCESS", "message": message}

## AWS SNS (Simple Notification Service)

O SNS é um serviço de publicação/assinatura (pub/sub) usado para enviar mensagens ou notificações.
Em um workflow automatizado, ele pode ser acionado pela Step Function após o término de um processamento.

Pode enviar mensagens para e-mails, endpoints HTTP/S, Lambda ou SQS.

É útil para alertas, notificações de status e integração entre sistemas.

## AWS SQS (Simple Queue Service)

O SQS é um serviço de mensageria assíncrona, ideal para desacoplar componentes do sistema.
Em uma Step Function, ele pode ser usado para armazenar mensagens de processamento ou coordenar tarefas assíncronas.

Garante entrega de mensagens com confiabilidade;

Suporta filas FIFO e Standard;

Permite escalabilidade e tolerância a falhas.

## Arquitetura do Workflow Automatizado

O workflow proposto realiza as seguintes etapas:

Recebe um evento inicial (ex: item a ser processado);

Executa uma Lambda que processa o item;

Envia uma notificação SNS confirmando a execução;

Publica uma mensagem na fila SQS para ser consumida por outro serviço;

Finaliza o fluxo com status de sucesso.

🔄 Diagrama Conceitual

stateDiagram-v2
    [*] --> LambdaTask
    LambdaTask --> NotifySNS
    NotifySNS --> SendSQS
    SendSQS --> [*]

🧰 Definição do Workflow (Step Functions JSON)

{
  "Comment": "Workflow automatizado com Lambda, SNS e SQS",
  "StartAt": "Processar Item",
  "States": {
    "Processar Item": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:ProcessItemLambda",
      "Next": "Enviar Notificação SNS"
    },
    "Enviar Notificação SNS": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Message": "Processamento concluído com sucesso!",
        "TopicArn": "arn:aws:sns:REGION:ACCOUNT_ID:TopicSucesso"
      },
      "Next": "Enviar Mensagem SQS"
    },
    "Enviar Mensagem SQS": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sqs:sendMessage",
      "Parameters": {
        "QueueUrl": "https://sqs.REGION.amazonaws.com/ACCOUNT_ID/FilaProcessamento",
        "MessageBody": {
          "Resultado": "SUCCESS",
          "Timestamp.$": "$$.State.EnteredTime"
        }
      },
      "End": true
    }
  }
}

🧩 Execução e Testes

Implante as funções e filas na AWS (via Console ou CloudFormation);

Crie a Step Function e importe o JSON acima;

Execute uma instância do workflow com um evento de teste;

Monitore a execução no painel visual do Step Functions;

Verifique os logs no CloudWatch, SNS e SQS.

## Insights e Aprendizados

O Step Functions simplifica a orquestração de múltiplos serviços AWS sem necessidade de código adicional de controle de fluxo.

A integração com Lambda, SNS e SQS torna os fluxos escaláveis, resilientes e desacoplados.

A representação visual facilita o debugging e o monitoramento.

A utilização de Infrastructure as Code (IaC) permite reproduzir e versionar o fluxo facilmente.

## Conclusão

Este desafio consolidou o entendimento prático sobre workflows automatizados na AWS, demonstrando como Step Functions, combinadas com Lambda, SNS e SQS, podem criar pipelines serverless, confiáveis e fáceis de manter.
