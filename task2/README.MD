


kubectl create namespace prod
helm install mykafka bitnami/kafka  --namespace prod --set replicaCount=5,auth.clientProtocol=plaintext,allowPlaintextListener=true,advertisedListeners=PLAINTEXT://:9092,listenerSecurityProtocolMap=PLAINTEXT:PLAINTEXT,interBrokerListenerName="PLAINTEXT",image.tag=3.5.2


kubectl create dev
helm install mykafka bitnami/kafka  --namespace dev --set replicaCount=1,auth.clientProtocol=plaintext,allowPlaintextListener=true,advertisedListeners=PLAINTEXT://:9092,listenerSecurityProtocolMap=SASL_PLAINTEXT:SASL_PLAINTEXT,interBrokerListenerName="SASL_PLAINTEXT",image.tag=latest