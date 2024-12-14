# Hospital

O objetivo é substituir suas planilhas antigas por um banco de dados moderno. Desenvolvendo uma estrutura completa que registra médicos, pacientes, consultas, convênios e receitas médicas. O sistema centraliza e organiza os dados, permitindo acesso e atualizações rápidas e seguras. Com isso, melhorando a eficiência do hospital, garantindo maior precisão no gerenciamento das informações clínicas.

## Estrutura

**Consultas**: Representa um atendimento realizado por um médico a um paciente.
* data
* id do médico
* id do paciente
* convênio
* valor
* diagnóstico
* receita

**Internações**: Representa o processo de admissão de um paciente para tratamento hospitalar.
* id do paciente
* id do médico
* datas
* procedimentos
* enfermeiros
* quarto

**Médicos**: Representa os médicos cadastrados no sistema.
* nome
* data de nascimento
* especialidades
* contatos
* documentos

**Enfermeiros**: Representa os enfermeiros cadastrados no sistema.
* nome
* documentos

![Diagrama](https://github.com/user-attachments/assets/a4af8597-ac5b-4530-a96d-c3288d1d843c)


## Consultas

1. Todos os dados e o valor médio das consultas do ano de 2020 e das que foram feitas sob convênio.

<pre>db.consultas.aggregate([
{ 
  $match: {
  $or: [
{ "data": { $gte: ISODate("2020-01-01"), $lte: ISODate("2020-12-31") } },
{ "conveniada": true }]
}
},
{ $group: {
  "_id": null,
  "media": { $avg: "$valor" },
  "consultas": { $push: "$$ROOT" } }
}
])</pre>

2. Todos os dados das internações que tiveram data de alta maior que a data prevista para a alta.

<pre>db.internacoes.find(
{ 
  $expr: { $gt: ["$data_efetiva_alta", 
  "$data_prevista_alta"] }
})</pre>


3. Receituário completo da primeira consulta registrada com receituário associado.

<pre>db.consultas.find(
{ 
  "receita": { $exists: true, $ne: null } },
{
  "data": 1,
  "receita": 1
})
.sort({ "data": 1 })
.limit(1)</pre>

4. Todos os dados da consulta de maior valor e também da de menor valor (ambas as consultas não foram realizadas sob convênio).

<pre>db.consultas.aggregate([
{ 
  $match: {
  "conveniada": false
}
},
{ $facet: {
  menor: [
{ $sort: { "valor": 1 } },
{ $limit: 1 }],
  maior: [
{ $sort: { "valor": -1 } },
{ $limit: 1 }] }
}
])</pre>

5. Todos os dados das internações em seus respectivos quartos, calculando o total da internação a partir do valor de diária do quarto e o número de dias entre a entrada e a alta.

<pre>db.internacoes.aggregate([
{
  $addFields: {
  "dias_internacao": {
  $ceil: {
  $divide: [
{ $subtract: ["$data_efetiva_alta", "$data_entrada"] },
  1000 * 60 * 60 * 24 ]} }
}
},
{ $addFields: {
  total_internacao: {
  $multiply: ["$dias_internacao", "$valor_diario"]} }
}
])</pre>

6. Data, procedimento e número de quarto de internações em quartos do tipo “apartamento”.

<pre>db.internacoes.aggregate([
{
  $match: { 
  "quarto.tipo": "Apartamento" }
},
{ $project: {
  "_id": 0,
  "data_entrada": 1,
  "procedimentos": 1,
  "quarto.numero": 1 }
}
])</pre>

7. Nome do paciente, data da consulta e especialidade de todas as consultas em que os pacientes eram menores de 18 anos na data da consulta e cuja especialidade não seja “pediatria”, ordenando por data de realização da consulta.

<pre>db.consultas.aggregate([
{
  $lookup: {
  "from": "pacientes", 
  "localField": "paciente_id", 
  "foreignField": "_id", 
  "as": "paciente_info" }
},
{
  $unwind: "$paciente_info" 
},
{
  $addFields: {
  "idadePaciente": {
  $dateDiff: {
  "startDate": "$paciente_info.data_nascimento", 
  "endDate": "$data",
  "unit": "year" }} }
},
{
  $match: {
  "idadePaciente": { $lt: 18 },
  "especialidade_buscada": { $ne: "pediatria" } }
},
{
  $project: {
  "nomePaciente": "$paciente_info.nome", 
  "dataConsulta": "$data", 
  "especialidade": "$especialidade_buscada",
  "idade": "$idadePaciente" }
}
])
  .sort({
  dataConsulta: 1
})</pre>

8. Nome do paciente, nome do médico, data da internação e procedimentos das internações realizadas por médicos da especialidade “gastroenterologia”, que tenham acontecido em “enfermaria”.

<pre>db.internacoes.aggregate([
{
  $lookup: {
  "from": "pacientes", 
  "localField": "paciente_id", 
  "foreignField": "_id", 
  "as": "paciente_info" }
},
{
  $unwind: "$paciente_info" 
},
{
  $lookup: {
  "from": "medicos", 
  "localField": "medico_id", 
  "foreignField": "_id", 
  "as": "medico_info" }
},
{
  $unwind: "$medico_info" 
}, 
{
  $match: {
  "medico_info.especialidades": "gastroenterologia",
  $or: [
{ "quarto_numero": 103 },
{ "quarto_numero": 104 }] }
},
{
  $project: {
  "_id": 0,
  "nome_paciente": "$paciente_info.nome",
  "nome_medico": "$medico_info.nome",
  "data_internacao": "$data_entrada",
  "procedimentos": "$procedimentos" }
}
])</pre>

9. Os nomes dos médicos, seus CRMs e a quantidade de consultas que cada um realizou.

<pre>db.consultas.aggregate([
{
  $group: {
  "_id": "$medico_id", 
  "totalConsultas": { $sum: 1 } }
},
{
  $lookup: {
  "from": "medicos", 
  "localField": "_id", 
  "foreignField": "_id", 
  "as": "medico" }
},
{
  $unwind: "$medico" 
},
{
  $project: {
  "_id": 0,
  "nome": "$medico.nome", 
  "CRM": "$medico.documentos.CRM", 
  "totalConsultas": 1 }
}
])</pre>

10. Todos os médicos que tenham "Gabriel" no nome.

<pre>db.medicos.find(
{
  "nome": /Gabriel/i
})</pre>

11. Os nomes, CORENs e número de internações de enfermeiros que participaram de mais de uma internação.

<pre>db.internacoes.aggregate([
{ 
  $unwind: "$enfermeiro_id" 
}, 
{
  $group: {
  "_id": "$enfermeiro_id",
  "totalInternacoes": { $sum: 1 } }
},
{ 
   $match: {
   "totalInternacoes": { $gte: 2 } } 
},
{
   $lookup: {
   "from": "enfermeiros", 
   "localField": "_id", 
   "foreignField": "_id", 
   "as": "enfermeiro" }
},
{
   $unwind: "$enfermeiro"
},
{
   $project: {
   "_id": 0,
   "enfermeiro_nome": "$enfermeiro.nome",
   "enfermeiro_CRE": "$enfermeiro.documentos.CRE",
   "totalInternacoes": 1 }
}
])</pre>
