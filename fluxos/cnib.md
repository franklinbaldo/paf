```mermaid
flowchart TD


subgraph rotina_cnib
  verifica_cnib{Decretado CNIB?}
  verifica_cnib --> |Sim| cnib_decretado
  verifica_cnib --> |Não| tipo_divida{Tipo Divida?}
  tipo_divida --> |Tributária| peticiona_cnib
  tipo_divida --> |Não Tributária| verificar_prescricao
  peticiona_cnib --> verifica_cnib
  peticiona_cnib --> cnib_decretado 
  cnib_decretado --> |Depois de 1 ano| verificar_prescricao
end```