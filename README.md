# Banco de Dados de Hospital

Bem-vindo ao projeto de banco de dados hospitalar utilizando **MongoDB**! Este banco de dados não-estruturado foi projetado para gerenciar informações essenciais relacionadas à administração de um hospital.

## 🔍 Funcionalidades

O banco de dados contém as seguintes coleções:

- **Consultas:** Informações sobre agendamentos e atendimentos médicos.
- **Convênios:** Detalhes dos convênios aceitos pelo hospital.
- **Enfermeiros:** Dados dos enfermeiros, incluindo turnos e especializações.
- **Internações:** Registros de pacientes internados e seus respectivos períodos.
- **Médicos:** Informações sobre os médicos, especialidades e horários de atendimento.
- **Pacientes:** Dados dos pacientes, histórico médico e contatos de emergência.
- **Quartos:** Detalhes sobre ocupação e tipos de acomodações disponíveis.

## 🎯 Objetivo do Projeto

O objetivo deste projeto é criar um sistema eficiente para armazenar e gerenciar dados hospitalares. Utilizando o **MongoDB**, é possível aproveitar a flexibilidade do armazenamento de dados não-estruturados, permitindo futuras expansões e integrações.

## 🗂 Estrutura do Banco de Dados

Exemplo de documento na coleção **Pacientes**:

```json
{
  "nome": "João Silva",
  "idade": 45,
  "contato": "(11) 98765-4321",
  "historico_medico": ["Diabetes", "Hipertensão"],
  "internacoes": [
    {
      "data": "2024-11-01",
      "quarto": 12,
      "motivo": "Cirurgia"
    }
  ]
}
```

## 🚀 Como Executar

1. Certifique-se de ter o **MongoDB** instalado.
2. Clone este repositório:
   ```bash
   git clone https://github.com/seu-usuario/seu-repositorio.git
   ```
3. Importe os dados para o MongoDB:
   ```bash
   mongoimport --db hospital --collection pacientes --file pacientes.json
   ```
4. Repita o processo para as demais coleções.

## 🤝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir uma **issue** ou enviar um **pull request** com melhorias ou novas funcionalidades.


---

Desenvolvido com ❤️ por Victor de Curtis Oliveira Zampella
