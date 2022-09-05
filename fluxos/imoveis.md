```mermaid
flowchart TD

subgraph rotina_arisp
  pesquisa_arisp[/Pesquisa Arisp/]
  pesquisa_arisp --> |Positivo| peticiona_penhora_imovel
  pesquisa_arisp --> |Negativa| pesquisa_veiculos
end
```