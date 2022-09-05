```mermaid
flowchart TD

subgraph rotina_cnib
  cnib
end

subgraph rotina_veiculos
  pesquisa_veiculos --> |Positivo| peticiona_penhora_veiculo
  pesquisa_veiculos --> |Negativa| cnib
end


```