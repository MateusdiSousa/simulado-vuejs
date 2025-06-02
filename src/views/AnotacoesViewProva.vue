<script setup lang="ts">
import { onMounted, ref } from "vue"
import axios from 'axios'
import CustomSelect from "../components/CustomSelect.vue"
import DataTable from "../components/DataTable.vue"


interface AnotacoesDto {
  id?: number
  texto: string
  dataHora: Date
  usuario: Usuario
}

interface Usuario {
  id?: number
  nome: string
  autorizacoes: Autorizacao[]
}

interface Autorizacao {
  id?: number
  nome: string
}

function getAntiguidade(dataHora : Date) {
  let meses : number = 0
  const dataHoraDocumentos : Date= new Date(dataHora)
  const dataHoraAtual : Date= new Date()

  if (dataHoraAtual.getMonth() >= dataHoraDocumentos.getMonth() ) {
    meses += dataHoraAtual.getMonth() - dataHoraDocumentos.getMonth()
    meses += (dataHoraAtual.getFullYear() - dataHoraDocumentos.getFullYear()) *12
  }

  if (dataHoraAtual.getMonth() < dataHoraDocumentos.getMonth() ) {
    meses += ((dataHoraAtual.getFullYear() - dataHoraDocumentos.getFullYear()) * 12)
    meses -= dataHoraDocumentos.getMonth() - dataHoraAtual.getMonth()
  }

  return meses
}

const anotacoes = ref<AnotacoesDto[]>([])
const usuarios = ref<Usuario[]>([])
const usuarioSelecionado = ref<Usuario>({})
const usuarioSelecionado2 = ref<Usuario>({})
const dataHoraInput = ref<Date | null>(null)
const anotacaoInput = ref<string>("")
const erro = ref<string>('')
const vocabulario = ref<string>("")
const tableHeaders = [
  { text: 'ID', value: 'id' },
  { text: 'Texto', value: 'texto' },
  { text: 'Antiguidade (meses)', value: 'dataHora',
    formatter: (value: Date) => getAntiguidade(value)
  },
  { text: 'Usuário', value: 'usuario', formatter: (value: Usuario) => value.nome }
]


async function atualizar() {
  try {
    const resp = await axios.get('usuario')
    usuarios.value = resp.data
    erro.value = ''
  } catch (e) {
    erro.value = (e as Error).message
  }
}

async function getAnotacoes() {
  const resp = await axios.get("anotacao")
  anotacoes.value = resp.data
}

onMounted(async () => {
  await atualizar()
  await getAnotacoes()
})


async function adicionarAnotacao() {
  if (anotacaoInput.value == "" || dataHoraInput.value == null || usuarioSelecionado.value == {}) {
    alert("Todos os campos devem ser preenchidos")
    return
  }

  const body : AnotacoesDto = {
    texto: anotacaoInput.value,
    dataHora: dataHoraInput.value,
    usuario: usuarioSelecionado.value
  }

  const resp = await axios.post("anotacao", body)
  if (resp.status == 200 || resp.status == 201) {
    alert("Anotação criada com suceeso")
  }
  await getAnotacoes()
}

async function procurarPalavra() {
  const resp = await axios.get(`anotacao/${usuarioSelecionado2.value.nome}/${vocabulario.value}`)
  if (resp.data == []) {
    anotacoes.value = []
    return
  }
  anotacoes.value = resp.data
}

</script>

<template>
    <DataTable
      :items="anotacoes"
      :headers="tableHeaders"
      no-data-message="Nenhuma anotação encontrada"
    />



    <form action="submit" @submit.prevent="adicionarAnotacao()">
      <h2>Criar Anotação</h2>
      <input type="text" placeholder="anotacao" v-model="anotacaoInput">
      <input type="datetime-local" placeholder="data hora" v-model="dataHoraInput">
      <CustomSelect
        v-model="usuarioSelecionado"
        :items="usuarios"
        label="Selecione o usuário: "
        display-property="nome"
        key-property="id"
        >
      </CustomSelect>
      <button type="submit">CriarAnotacao</button>
    </form>


    <form action="submit" @submit.prevent="procurarPalavra()">
      <h2>Pesquisar vocabulario</h2>
      <input type="text" placeholder="Digite a palavra que deseja procurar" v-model="vocabulario">
      <CustomSelect
        v-model="usuarioSelecionado2"
        :items="usuarios"
        label="Selecione o usuário: "
        display-property="nome"
        key-property="id">
      </CustomSelect>
      <button type="submit"> Procurar </button>
    </form>
</template>

<style>
form{
  align-items: center;
  display: flex;
  flex-direction: column;
  justify-items: center;
}

input{
  padding: 5px;
  margin-bottom: 5px;
  width: 50vh;
}

</style>
