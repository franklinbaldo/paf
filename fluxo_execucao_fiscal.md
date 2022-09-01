# paf


```mermaid
flowchart TD

subgraph rotina_pagamento[Rotina pagamento]
  noticia_pagamento(Notícia de pagamento) --> verifica_sitafe{Verifica sitafe}
  verifica_sitafe --> |Todas CDA's quitadas| verifica_honorarios{Verifica honorarios}
  verifica_sitafe --> |Saldo não pago| atos_constritivos
  verifica_honorarios --> |Honorários quitados| execucao_quitada
  execucao_quitada --> peticiona_extincao_por_pagamento[Petiona extinção por pagamento]
end

subgraph atos_constritivos 
  inicio_prescricao(Início da prescrição) --> verifica_devedor{Tipo devedor?}
  verifica_devedor --> |CPF| devedor_cpf((CPF))
  devedor_cpf --> devedor_ativo
  verifica_devedor --> |CNPJ| devedor_cnpj
  subgraph rotina_prescricao
    verificar_prescricao{Está prescrito?}
    verificar_prescricao --> |Sim| peticiona_prescricao_intercorrente
    verificar_prescricao --> |Não| verifica_devedor
  end
end

subgraph rotina_cnib
  verifica_cnib{Decretado CNIB?}
  verifica_cnib --> |Sim| cnib_decretado
  verifica_cnib --> |Não| tipo_divida{Tipo Divida?}
  tipo_divida --> |Tributária| peticiona_cnib
  tipo_divida --> |Não Tributária| verificar_prescricao
  peticiona_cnib --> verifica_cnib
  peticiona_cnib --> cnib_decretado 
  cnib_decretado --> |Depois de 1 ano| verificar_prescricao
end

subgraph rotina_veiculos
  pesquisa_veiculos --> |Positivo| peticiona_penhora_veiculo
  pesquisa_veiculos --> |Negativa| verifica_cnib
end

subgraph rotina_arisp
  pesquisa_arisp[/Pesquisa Arisp/]
  pesquisa_arisp --> |Positivo| peticiona_penhora_imovel
  pesquisa_arisp --> |Negativa| pesquisa_veiculos
end

subgraph rotina_sisbajud
  devedor_ativo -->  verifica_viabilidade_sisbajud{Sisbajud viável?}
  verifica_viabilidade_sisbajud --> |Anterior positivo ou primeiro sisbajud| peticiona_sisbajud
  verifica_viabilidade_sisbajud --> |Anterior negativo| sisbajud_inviavel(Sisbajud inviável)
  sisbajud_inviavel --> pesquisa_arisp
  peticiona_sisbajud --> sisbajud_positivo(Sisbajud positivo)
  sisbajud_positivo --> prazo_embargos{Embargos do devedor?}
  prazo_embargos --> | Sem embargos | peticiona_levantamento --> levantamento_efetuado
  levantamento_efetuado --> vincular_dare
  vincular_dare --> noticia_pagamento
  prazo_embargos --> | Houve embargos | embargos_do_devedor_apresentado
end

subgraph rotina_excecao
  inicio_prescricao --> excecao_preexecutividade
  excecao_preexecutividade --> analise_de_excecao{Análise da Exceção}
  analise_de_excecao --> |Tem fundamento| corrige_cda
  analise_de_excecao --> |Sem fundamento| peticiona_resposta_excecao
  corrige_cda --> peticiona_resposta_excecao
  peticiona_resposta_excecao --> verifica_devedor
end


subgraph rotina_cnpj
  devedor_cnpj((CNPJ)) -->  verifica_situacao_cnpj{Situação do CNPJ?}
  verifica_situacao_cnpj --> |Ativo na RFB e no Sintegra| cnpj_ativo(CNPJ ativo)
  verifica_situacao_cnpj --> |Inativo na RFB ou no Sintegra| cnpj_inativo(CNPJ inativo)
  cnpj_ativo --> verifica_ei{Empresário individual?}
  verifica_ei --> |Não| devedor_ativo
  verifica_ei --> |Sim| cnpj_ei_pje{CPF está no PJE?}
  cnpj_ei_pje --> |Sim| cnpj_ei_ok
  cnpj_ei_ok --> devedor_ativo
  cnpj_ei_pje --> |Não| peticiona_ei --> cnpj_ei_pje
end

subgraph rotina_redirecionamento
  cnpj_inativo -->  verificar_redirecionamentos{Já redirecionado para administradores?}
  verificar_redirecionamentos --> |Sim| demais_devedores_citados[Demais devedores citados]
  verificar_redirecionamentos --> |Não| peticionar_redirecionamento
  peticionar_redirecionamento  --> demais_devedores_citados
  demais_devedores_citados --> |Quanto aos demais| verifica_devedor
  demais_devedores_citados --> |Quanto ao CNPJ| verifica_cnib
end

subgraph rotina_citacao[Rotina citação]
  mandado_citacao{Mandado de citação}
  mandado_citacao --> |Positivo| parte_citada(Parte citada)
  mandado_citacao --> |Negativo| pesquisa_endereco{Pesquisa endereço}
  pesquisa_endereco --> |Encontrado novo endereço| peticiona_novo_endereco[Peticiona para informar novo endereço]
  peticiona_novo_endereco --> mandado_citacao
petiona_nova_modalidade[Peticiona nova modalidade: Mandado, Hora certa, Edital]
  pesquisa_endereco --> |Não encontrado novo endereço e enquanto não for citado| petiona_nova_modalidade
  petiona_nova_modalidade --> mandado_citacao
  parte_citada --> prazo_para_pagamento(5 dias para pagamento)
  prazo_para_pagamento --> |Notícia de pagamento| noticia_pagamento
  prazo_para_pagamento --> |Não pagou nem ofereceu garantia| inicio_prescricao
end

subgraph rotina_ajuizamento
  ajuizamento(Ajuizamento)
  ajuizamento(Ajuizamento) -->despacho_inicial{Despacho incial}
  despacho_inicial --> |Petição está OK| mandado_citacao
  despacho_inicial --> |Defeito na inicial| peticiona_correcao 
  peticiona_correcao --> despacho_inicial
end

