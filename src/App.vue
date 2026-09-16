<template>
  <div class="min-h-screen bg-gray-900 text-gray-100 flex flex-col items-center p-6">
    <header class="text-center my-8">
      <p class="text-sm font-semibold uppercase tracking-[0.2em] text-emerald-400">Planejamento financeiro</p>
      <h1 class="text-3xl font-extrabold text-white tracking-tight mt-2">{{ projeto.nome || 'Configure seu projeto' }}</h1>
      <p class="text-gray-400 mt-2">Preencha os dados abaixo para iniciar a simulação.</p>
    </header>

    <main class="w-full max-w-6xl bg-gray-800 rounded-2xl shadow-xl border border-gray-700 p-6">
      <nav v-if="projetoConfigurado" class="grid grid-cols-2 sm:grid-cols-5 gap-2 mb-6" aria-label="Etapas do projeto">
        <button
          v-for="etapa in etapas"
          :key="etapa.id"
          type="button"
          @click="abrirEtapa(etapa.id)"
          :disabled="!etapaDisponivel(etapa.id) && etapa.id !== paginaAtual"
          class="rounded-lg border px-3 py-2 text-sm font-semibold transition-colors disabled:cursor-not-allowed disabled:opacity-50"
          :class="paginaAtual === etapa.id
            ? 'border-emerald-500 bg-emerald-500 text-gray-900'
            : etapaDisponivel(etapa.id)
              ? 'border-gray-600 bg-gray-700 text-gray-300 hover:border-emerald-500 hover:text-white'
              : 'border-gray-700 bg-gray-800 text-gray-500'"
        >
          {{ etapa.nome }}
        </button>
      </nav>

      <div
        v-if="confirmacaoLimpezaPagina"
        class="mb-5 rounded-lg border border-red-500/40 bg-red-950/30 p-4"
        role="alertdialog"
        aria-labelledby="titulo-confirmacao-limpeza-pagina"
      >
        <h3 id="titulo-confirmacao-limpeza-pagina" class="font-semibold text-red-200">
          {{ confirmacaoLimpezaPagina === 'dados-projeto' ? 'Limpar dados do projeto?' : 'Limpar dados desta página?' }}
        </h3>
        <p class="mt-1 text-sm text-gray-300">
          {{ confirmacaoLimpezaPagina === 'dados-projeto'
            ? 'Todos os dados preenchidos até agora serão apagados'
            : `Somente os dados de ${nomePaginaLimpeza} serão apagados.` }}
        </p>
        <div class="mt-3 flex gap-3">
          <button
            type="button"
            class="rounded-lg bg-red-500 px-4 py-2 text-sm font-semibold text-white hover:bg-red-600 transition-colors"
            @click="limparPaginaLocal"
          >
            Sim, limpar página
          </button>
          <button
            type="button"
            class="rounded-lg border border-gray-600 px-4 py-2 text-sm font-semibold text-gray-300 hover:border-gray-400 transition-colors"
            @click="confirmacaoLimpezaPagina = ''"
          >
            Cancelar
          </button>
        </div>
      </div>

      <form v-if="!projetoConfigurado || paginaAtual === 'projeto'" @submit.prevent="configurarProjeto" class="space-y-5">
        <div>
          <h2 class="text-xl font-bold text-emerald-300">Dados do projeto</h2>
          <p class="text-sm text-gray-400 mt-1">Informe o desenvolvimento e, se houver, o período de garantia.</p>
        </div>

        <div class="border-b border-gray-700 pb-5">
          <label
            for="importar-cadastro-csv"
            class="inline-block cursor-pointer rounded-lg bg-emerald-500 px-4 py-2 text-sm font-semibold text-gray-900 hover:bg-emerald-600 transition-colors"
          >
            Importar arquivo CSV
          </label>
          <input
            id="importar-cadastro-csv"
            type="file"
            accept=".csv,text/csv"
            class="sr-only"
            @change="importarCadastroCsv"
          />
          <button
            type="button"
            class="ml-3 rounded-lg border border-red-500/60 px-4 py-2 text-sm font-semibold text-red-300 hover:bg-red-500/10 transition-colors"
            @click="abrirConfirmacaoLimpezaPagina('projeto')"
          >
            Limpar página
          </button>
          <button
            type="button"
            class="ml-3 rounded-lg border border-amber-500/60 px-4 py-2 text-sm font-semibold text-amber-300 hover:bg-amber-500/10 transition-colors"
            @click="abrirConfirmacaoLimpezaPagina('dados-projeto')"
          >
            Limpar dados do projeto
          </button>
        </div>

        <div>
          <label for="nome-projeto" class="block text-sm font-medium text-gray-300 mb-1">Nome do projeto (opcional)</label>
          <input
            id="nome-projeto"
            v-model="projeto.nome"
            type="text"
            placeholder="Ex.: Sistema financeiro"
            class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white placeholder:text-gray-500 focus:outline-none focus:border-emerald-500 transition-colors"
          />
        </div>

        <div>
          <label for="tempo-desenvolvimento" class="block text-sm font-medium text-gray-300 mb-1">Tempo de desenvolvimento (meses)</label>
          <input
            id="tempo-desenvolvimento"
            v-model.number="projeto.tempoMeses"
            type="number"
            min="1"
            required
            class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
          />
        </div>

        <div>
          <label for="inicio-previsto" class="block text-sm font-medium text-gray-300 mb-1">Início previsto</label>
          <input
            id="inicio-previsto"
            v-model="projeto.inicioPrevisto"
            type="date"
            lang="pt-BR"
            required
            class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
          />
        </div>

        <fieldset class="border-t border-gray-700 pt-5 space-y-4">
          <legend class="text-sm font-semibold text-emerald-300">Garantia e encerramento</legend>

          <div>
            <label for="termino-desenvolvimento" class="block text-sm font-medium text-gray-300 mb-1">Último mês de desenvolvimento</label>
            <div
              id="termino-desenvolvimento"
              class="w-full bg-gray-700/60 border border-gray-600 rounded-lg px-3 py-2 text-gray-200"
            >
              {{ mesParaExibicao(mesFinalDesenvolvimento) || 'Informe o início previsto' }}
            </div>
          </div>

          <label class="flex items-center gap-3 text-sm text-gray-300 cursor-pointer">
            <input
              v-model="projeto.temGarantia"
              type="checkbox"
              class="h-4 w-4 accent-emerald-500"
            />
            Possui garantia e encerramento
          </label>

          <div v-if="projeto.temGarantia">
            <label for="meses-garantia" class="block text-sm font-medium text-gray-300 mb-1">Meses de garantia e encerramento</label>
            <input
              id="meses-garantia"
              v-model.number="projeto.mesesGarantia"
              type="number"
              min="1"
              required
              class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
            />
          </div>

          <div v-if="projeto.temGarantia">
            <label for="data-garantia-encerramento" class="block text-sm font-medium text-gray-300 mb-1">Garantia e encerramento</label>
            <div
              id="data-garantia-encerramento"
              class="w-full bg-gray-700/60 border border-gray-600 rounded-lg px-3 py-2 text-gray-200"
            >
              {{ dataGarantiaEncerramento || 'Informe os meses de garantia' }}
            </div>
          </div>
        </fieldset>

        <button
          type="submit"
          :disabled="!formularioValido"
          class="w-full font-bold py-3 px-4 rounded-lg transition-colors"
          :class="formularioValido
            ? 'bg-emerald-500 hover:bg-emerald-600 text-gray-900'
            : 'bg-gray-600 text-gray-400 cursor-not-allowed'"
        >
          {{ projetoConfigurado ? 'Salvar alterações e voltar para a equipe' : 'Continuar para a simulação' }}
        </button>
      </form>

      <section v-else-if="paginaAtual === 'equipe'">
        <div class="mb-6 flex items-start justify-between gap-4">
          <div>
            <p class="text-sm font-semibold uppercase tracking-wider text-emerald-400">Próxima etapa</p>
            <h2 class="text-xl font-bold text-white mt-1">Equipe Financeira</h2>
            <p class="text-sm text-gray-400 mt-1">Cadastre os profissionais que participarão do projeto.</p>
          </div>
          <button type="button" class="rounded-lg border border-red-500/60 px-3 py-2 text-sm font-semibold text-red-300 hover:bg-red-500/10 transition-colors" @click="abrirConfirmacaoLimpezaPagina('equipe')">
            Limpar página
          </button>
        </div>

        <div class="space-y-4">
          <div
            v-for="(membro, indice) in equipeFinanceira"
            :key="indice"
            class="space-y-3"
          >
            <div class="grid grid-cols-1 sm:grid-cols-[auto_1.1fr_1fr_1.8fr_1.1fr_1.1fr_1.2fr_1.2fr_1.2fr] gap-4 items-end">
              <button
                type="button"
                @click="removerMembro(indice)"
                class="h-10 w-10 rounded-lg text-xl font-semibold text-red-400 hover:bg-red-500/20 hover:text-red-300 transition-colors"
                :aria-label="`Excluir pesquisador ${indice + 1}`"
                :title="`Excluir pesquisador ${indice + 1}`"
              >
                ×
              </button>

              <div>
                <label :for="`titulacao-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Titulação</label>
                <select
                  :id="`titulacao-${indice}`"
                  v-model="membro.titulacao"
                  @click.stop
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500"
                >
                  <option value="" disabled>Selecione</option>
                  <option value="Técnico">Técnico</option>
                  <option value="Graduação">Graduação</option>
                  <option value="Especialização">Especialização</option>
                  <option value="Mestrado">Mestrado</option>
                  <option value="Doutorado">Doutorado</option>
                </select>
              </div>

              <div>
                <label :for="`cpf-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">CPF</label>
                <input
                  :id="`cpf-${indice}`"
                  :value="membro.cpf"
                  @input="formatarCpf(membro, $event)"
                  type="text"
                  inputmode="numeric"
                  maxlength="14"
                  :class="[
                    'w-full bg-gray-700 border rounded-lg px-3 py-2 text-white focus:outline-none transition-colors',
                    membro.cpf && !cpfValido(membro.cpf)
                      ? 'border-red-500 focus:border-red-500'
                      : 'border-gray-600 focus:border-emerald-500'
                  ]"
                />
                <p v-if="membro.cpf && !cpfValido(membro.cpf)" class="text-xs text-red-400 mt-1">Informe um CPF válido.</p>
              </div>

              <div>
                <label :for="`nome-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Nome</label>
                <input
                  :id="`nome-${indice}`"
                  v-model="membro.nome"
                  type="text"
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>

              <div>
                <label :for="`salario-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Salário</label>
                <input
                  :id="`salario-${indice}`"
                  v-model="membro.salario"
                  type="number"
                  min="0"
                  step="0.01"
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>

              <div>
                <label :for="`meses-participacao-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Meses de participação</label>
                <input
                  :id="`meses-participacao-${indice}`"
                  :value="membro.mesesParticipacao"
                  @input="limitarMesesParticipacao(membro, $event)"
                  type="number"
                  min="1"
                  :max="mesesDesenvolvimento"
                  required
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
                <p v-if="membro.erroMesesParticipacao" class="text-xs text-red-400 mt-1" role="alert">
                  {{ membro.erroMesesParticipacao }}
                </p>
              </div>

              <div>
                <label :for="`inicio-participacao-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Início (mês/ano)</label>
                <MonthPicker
                  :id="`inicio-participacao-${indice}`"
                  :model-value="membro.inicioParticipacao"
                  :min="mesInicialProjeto"
                  :max="mesMaximoInicioParticipacao(membro)"
                  @update:model-value="atualizarInicioEquipe(membro, $event)"
                />
              </div>

              <div>
                <label :for="`fim-participacao-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Fim (mês/ano)</label>
                <div :id="`fim-participacao-${indice}`" class="w-full bg-gray-700/60 border border-gray-600 rounded-lg px-3 py-2 text-gray-200">
                  {{ mesParaExibicao(calcularFimParticipacao(membro)) || 'Informe o início da participação' }}
                </div>
              </div>

              <div>
                <label :for="`horas-semanais-${indice}`" class="block text-xs font-medium text-gray-400 mb-1">Horas semanais por pesquisador</label>
                <input
                  :id="`horas-semanais-${indice}`"
                  v-model.number="membro.horasSemanais"
                  type="number"
                  min="1"
                  step="0.5"
                  required
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>
            </div>

            <button
              v-if="indice === equipeFinanceira.length - 1"
              type="button"
              :disabled="!linhaEquipePreenchida(membro)"
              @click="adicionarMembro"
              class="h-10 w-full rounded-lg text-xl font-semibold transition-colors"
              :class="linhaEquipePreenchida(membro)
                ? 'bg-emerald-500 text-gray-900 hover:bg-emerald-600'
                : 'bg-gray-700 text-gray-500 cursor-not-allowed'"
              :aria-label="`Adicionar membro abaixo da linha ${indice + 1}`"
            >
              +
            </button>
          </div>
        </div>

        <button
          type="button"
          @click="paginaAtual = 'alunos'"
          :disabled="!equipePreenchida"
          class="w-full mt-6 font-bold py-3 px-4 rounded-lg transition-colors"
          :class="equipePreenchida
            ? 'bg-emerald-500 hover:bg-emerald-600 text-gray-900'
            : 'bg-gray-600 text-gray-400 cursor-not-allowed'"
        >
          Continuar para o cadastro de alunos
        </button>
      </section>

      <section v-else-if="paginaAtual === 'alunos'">
        <div class="mb-6 flex items-start justify-between gap-4">
          <div>
            <p class="text-sm font-semibold uppercase tracking-wider text-emerald-400">Próxima etapa</p>
            <h2 class="text-xl font-bold text-white mt-1">Cadastro de alunos</h2>
            <p class="text-sm text-gray-400 mt-1">Cadastre os alunos participantes do projeto.</p>
          </div>
          <button type="button" class="rounded-lg border border-red-500/60 px-3 py-2 text-sm font-semibold text-red-300 hover:bg-red-500/10 transition-colors" @click="abrirConfirmacaoLimpezaPagina('alunos')">
            Limpar página
          </button>
        </div>

        <div class="space-y-4">
          <div v-for="(aluno, indice) in alunos" :key="aluno.id" class="space-y-3">
            <div class="grid grid-cols-1 sm:grid-cols-[auto_1.1fr_1.8fr_1.1fr_1.1fr_1.1fr_1.2fr] gap-4 items-end">
              <button
                type="button"
                @click="removerAluno(indice)"
                class="h-10 w-10 rounded-lg text-xl font-semibold text-red-400 hover:bg-red-500/20 hover:text-red-300 transition-colors"
                :aria-label="`Excluir ${aluno.nome}`"
                :title="`Excluir ${aluno.nome}`"
              >
                ×
              </button>

              <div>
                <label :for="`aluno-titulacao-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Qualificação</label>
                <select
                  :id="`aluno-titulacao-${aluno.id}`"
                  v-model="aluno.titulacao"
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500"
                >
                  <option value="" disabled>Selecione</option>
                  <option value="Técnico">Técnico</option>
                  <option value="Graduação">Graduação</option>
                  <option value="Especialização">Especialização</option>
                  <option value="Mestrado">Mestrado</option>
                  <option value="Doutorado">Doutorado</option>
                </select>
              </div>

              <div>
                <label :for="`aluno-nome-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Nome</label>
                <input
                  :id="`aluno-nome-${aluno.id}`"
                  v-model="aluno.nome"
                  type="text"
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>

              <div>
                <label :for="`aluno-meses-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Meses no projeto</label>
                <input
                  :id="`aluno-meses-${aluno.id}`"
                  :value="aluno.mesesParticipacao"
                  @input="limitarMesesAluno(aluno, $event)"
                  type="number"
                  min="1"
                  :max="maximoMesesAluno"
                  required
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
                <p v-if="aluno.erroMesesParticipacao" class="text-xs text-red-400 mt-1" role="alert">
                  {{ aluno.erroMesesParticipacao }}
                </p>
              </div>

              <div>
                <label :for="`aluno-inicio-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Início (mês/ano)</label>
                <MonthPicker
                  :id="`aluno-inicio-${aluno.id}`"
                  :model-value="aluno.inicioParticipacao"
                  :min="mesInicialProjeto"
                  :max="mesMaximoInicioParticipacao({ mesesParticipacao: aluno.mesesParticipacao })"
                  @update:model-value="atualizarInicioAluno(aluno, $event)"
                />
              </div>

              <div>
                <label class="block text-xs font-medium text-gray-400 mb-1">Fim (mês/ano)</label>
                <div class="w-full bg-gray-700/60 border border-gray-600 rounded-lg px-3 py-2 text-gray-200">
                  {{ mesParaExibicao(calcularFimAluno(aluno)) || 'Informe o início' }}
                </div>
              </div>

              <div>
                <label :for="`aluno-horas-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Horas semanais</label>
                <input
                  :id="`aluno-horas-${aluno.id}`"
                  v-model.number="aluno.horasSemanais"
                  type="number"
                  min="1"
                  step="0.5"
                  required
                  class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                />
              </div>
            </div>

            <div class="ml-14 max-w-xs">
              <label :for="`aluno-bolsa-${aluno.id}`" class="block text-xs font-medium text-gray-400 mb-1">Valor da bolsa (R$)</label>
              <input
                :id="`aluno-bolsa-${aluno.id}`"
                v-model="aluno.valorBolsa"
                type="number"
                min="0"
                step="0.01"
                placeholder="Será calculado futuramente"
                class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white placeholder:text-gray-500 focus:outline-none focus:border-emerald-500 transition-colors"
              />
            </div>

            <button
              v-if="indice === alunos.length - 1"
              type="button"
              :disabled="!linhaAlunoPreenchida(aluno)"
              @click="adicionarAluno"
              class="h-10 w-full rounded-lg text-xl font-semibold transition-colors"
              :class="linhaAlunoPreenchida(aluno)
                ? 'bg-emerald-500 text-gray-900 hover:bg-emerald-600'
                : 'bg-gray-700 text-gray-500 cursor-not-allowed'"
              aria-label="Adicionar aluno"
            >
              +
            </button>
          </div>
        </div>

        <button
          type="button"
          @click="paginaAtual = 'servicos'"
          :disabled="!alunosPreenchidos"
          class="w-full mt-6 font-bold py-3 px-4 rounded-lg transition-colors"
          :class="alunosPreenchidos
            ? 'bg-emerald-500 hover:bg-emerald-600 text-gray-900'
            : 'bg-gray-600 text-gray-400 cursor-not-allowed'"
        >
          Continuar para serviços e locomoção
        </button>
      </section>

      <section v-else-if="paginaAtual === 'servicos'">
        <div class="mb-6">
          <p class="text-sm font-semibold uppercase tracking-wider text-emerald-400">Próxima etapa</p>
          <h2 class="text-xl font-bold text-white mt-1">Serviços e locomoção</h2>
          <p class="text-sm text-gray-400 mt-1">Informe as aquisições de serviços, deslocamentos e outros custos relacionados ao projeto.</p>
        </div>

        <div class="flex flex-wrap items-center gap-3 mb-6">
          <label
            for="importar-orcamento-csv"
            class="cursor-pointer rounded-lg bg-emerald-500 px-4 py-2 text-sm font-semibold text-gray-900 hover:bg-emerald-600 transition-colors"
          >
            Importar orçamento CSV
          </label>
          <input
            id="importar-orcamento-csv"
            type="file"
            accept=".csv,text/csv"
            class="sr-only"
            @change="importarOrcamentoCsv"
          />
          <button
            type="button"
            class="rounded-lg border border-gray-600 px-4 py-2 text-sm font-semibold text-gray-300 hover:border-emerald-500 hover:text-white transition-colors"
            @click="baixarModeloCsv"
          >
            Baixar modelo CSV
          </button>
          <button
            type="button"
            class="rounded-lg border border-red-500/60 px-4 py-2 text-sm font-semibold text-red-300 hover:bg-red-500/10 transition-colors"
            @click="abrirConfirmacaoLimpezaPagina('servicos')"
          >
            Limpar página
          </button>
          <p v-if="mensagemImportacaoCsv" class="w-full text-sm text-emerald-300" role="status">
            {{ mensagemImportacaoCsv }}
          </p>
        </div>

        <div class="space-y-4">
          <div
            v-for="(item, indice) in servicosLocomocao"
            :key="item.id"
            class="grid grid-cols-1 sm:grid-cols-[auto_1fr_1.2fr_1.5fr_1fr] gap-4 items-end"
          >
            <button
              type="button"
              @click="removerServico(indice)"
              class="h-10 w-10 rounded-lg text-xl font-semibold text-red-400 hover:bg-red-500/20 hover:text-red-300 transition-colors"
              :aria-label="`Excluir item ${indice + 1}`"
              :title="`Excluir item ${indice + 1}`"
            >
              ×
            </button>

            <div>
              <label :for="`servico-tipo-${item.id}`" class="block text-xs font-medium text-gray-400 mb-1">Tipo</label>
              <select
                :id="`servico-tipo-${item.id}`"
                v-model="item.tipo"
                class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500"
              >
                <option value="" disabled>Selecione</option>
                <option value="diarias">Diárias</option>
                <option value="passagens-aereas">Passagens aéreas</option>
                <option value="servicos-terceiros">Serviços de terceiros</option>
                <option value="assinaturas">Assinaturas</option>
                <option value="bens-fisicos">Compra de bens físicos</option>
              </select>
            </div>

            <div>
              <label :for="`servico-descricao-${item.id}`" class="block text-xs font-medium text-gray-400 mb-1">Descrição</label>
              <input
                :id="`servico-descricao-${item.id}`"
                v-model="item.descricao"
                type="text"
                placeholder="Ex.: Transporte ou consultoria"
                class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white placeholder:text-gray-500 focus:outline-none focus:border-emerald-500 transition-colors"
              />
            </div>

            <div>
              <label :for="`servico-unidade-${item.id}`" class="block text-xs font-medium text-gray-400 mb-1">Unidade</label>
              <input
                :id="`servico-unidade-${item.id}`"
                v-model.number="item.unidade"
                type="number"
                min="1"
                step="1"
                class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
              />
            </div>

            <div>
              <label :for="`servico-valor-${item.id}`" class="block text-xs font-medium text-gray-400 mb-1">Valor (R$)</label>
              <input
                :id="`servico-valor-${item.id}`"
                v-model.number="item.valor"
                type="number"
                min="0"
                step="0.01"
                class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
              />
            </div>

            <button
              v-if="indice === servicosLocomocao.length - 1"
              type="button"
              :disabled="!linhaServicoPreenchida(item)"
              @click="adicionarServico"
              class="h-10 w-full sm:col-span-5 rounded-lg text-xl font-semibold transition-colors"
              :class="linhaServicoPreenchida(item)
                ? 'bg-emerald-500 text-gray-900 hover:bg-emerald-600'
                : 'bg-gray-700 text-gray-500 cursor-not-allowed'"
              aria-label="Adicionar serviço ou locomoção"
            >
              +
            </button>
          </div>
        </div>

        <button
          type="button"
          @click="paginaAtual = 'contrapartida'"
          :disabled="!servicosPreenchidos"
          class="w-full mt-6 font-bold py-3 px-4 rounded-lg transition-colors"
          :class="servicosPreenchidos
            ? 'bg-emerald-500 hover:bg-emerald-600 text-gray-900'
            : 'bg-gray-600 text-gray-400 cursor-not-allowed'"
        >
          Continuar para a contrapartida econômica
        </button>
      </section>

      <section v-else-if="paginaAtual === 'contrapartida'">
        <div class="mb-6">
          <p class="text-sm font-semibold uppercase tracking-wider text-emerald-400">Próxima etapa</p>
          <h2 class="text-xl font-bold text-white mt-1">Contrapartida Econômica</h2>
          <p class="text-sm text-gray-400 mt-1">Valores da equipe calculados automaticamente com base no cadastro da aba Equipe.</p>
        </div>

        <div class="overflow-x-auto rounded-xl border border-gray-700 bg-gray-900/40">
          <table class="min-w-full text-left text-sm text-gray-200">
            <thead class="bg-gray-800 text-gray-300">
              <tr>
                <th class="px-4 py-3 font-semibold">Nome da equipe</th>
                <th class="px-4 py-3 font-semibold">Salário</th>
                <th class="px-4 py-3 font-semibold">Meses de participação</th>
                <th class="px-4 py-3 font-semibold">Início (Mês/Ano)</th>
                <th class="px-4 py-3 font-semibold">Horas mensais</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(membro, indice) in linhasContrapartida" :key="`contrapartida-${indice}`" class="border-t border-gray-700 align-top">
                <td class="px-4 py-3">
                  <input
                    :value="membro.nome"
                    @input="atualizarNomeContrapartida(membro, $event)"
                    type="text"
                    placeholder="Nome"
                    :readonly="!membro.manualAdicao"
                    :disabled="!membro.manualAdicao"
                    class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white placeholder:text-gray-500 focus:outline-none focus:border-emerald-500 transition-colors disabled:cursor-not-allowed disabled:opacity-80"
                  />
                  <label class="mt-2 flex items-center gap-2 text-xs text-gray-300">
                    <input
                      v-model="membro.naoDocente"
                      type="checkbox"
                      class="h-4 w-4 accent-emerald-500"
                    />
                    Não é docente
                  </label>
                </td>
                <td class="px-4 py-3">
                  <input
                    :value="membro.salario"
                    @input="atualizarSalarioContrapartida(membro, $event)"
                    type="number"
                    min="0"
                    step="0.01"
                    placeholder="Salário"
                    :readonly="!membro.manualAdicao"
                    :disabled="!membro.manualAdicao"
                    class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white placeholder:text-gray-500 focus:outline-none focus:border-emerald-500 transition-colors disabled:cursor-not-allowed disabled:opacity-80"
                  />
                  <div v-if="membro.salario" class="mt-2 flex items-center justify-between gap-2 rounded-lg border border-emerald-500/20 bg-emerald-500/5 px-2 py-1 text-[11px] text-gray-300">
                    <span class="font-medium text-gray-400">Valor hora</span>
                    <span class="font-mono text-emerald-300">{{ obterValorHora(membro.salario, membro.naoDocente) }}</span>
                  </div>
                </td>
                <td class="px-4 py-3">
                  <input
                    :value="membro.mesesParticipacao"
                    @input="limitarMesesParticipacao(membro, $event)"
                    type="number"
                    min="1"
                    :max="mesesDesenvolvimento"
                    class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                  />
                  <p v-if="membro.erroMesesParticipacao" class="text-xs text-red-400 mt-1" role="alert">
                    {{ membro.erroMesesParticipacao }}
                  </p>
                </td>
                <td class="px-4 py-3">
                  <MonthPicker
                    :model-value="membro.inicioParticipacao"
                    :min="mesInicialProjeto"
                    :max="mesMaximoInicioParticipacao(membro)"
                    @update:model-value="atualizarInicioEquipe(membro, $event)"
                  />
                  <p v-if="membro.erroInicioParticipacao" class="text-xs text-red-400 mt-1" role="alert">
                    {{ membro.erroInicioParticipacao }}
                  </p>
                </td>
                <td class="px-4 py-3">
                  <input
                    v-model.number="membro.horasSemanais"
                    type="number"
                    min="1"
                    step="0.5"
                    class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
                  />
                  <div v-if="membro.horasSemanais" class="mt-2 flex items-center justify-between gap-2 rounded-lg border border-sky-500/20 bg-sky-500/5 px-2 py-1 text-[11px] text-gray-300">
                    <span class="font-medium text-gray-400">Horas semanais</span>
                    <span class="font-mono text-sky-300">{{ obterHorasSemanais(membro.horasSemanais) }}</span>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <button
          type="button"
          @click="adicionarMembroContrapartida"
          class="mt-4 h-10 w-full rounded-lg text-xl font-semibold bg-emerald-500 text-gray-900 hover:bg-emerald-600 transition-colors"
          aria-label="Adicionar membro manualmente"
        >
          +
        </button>

        <button
          type="button"
          @click="paginaAtual = 'simulador'"
          :disabled="!servicosPreenchidos"
          class="w-full mt-6 font-bold py-3 px-4 rounded-lg transition-colors"
          :class="servicosPreenchidos
            ? 'bg-emerald-500 hover:bg-emerald-600 text-gray-900'
            : 'bg-gray-600 text-gray-400 cursor-not-allowed'"
        >
          Continuar para a simulação
        </button>
      </section>

      <section v-else-if="paginaAtual === 'simulador'">
        <div class="flex items-start justify-between gap-4 mb-6">
          <div>
            <p class="text-sm font-semibold uppercase tracking-wider text-emerald-400">Projeto configurado</p>
            <h2 class="text-xl font-bold text-white mt-1">Simulador de Juros Compostos</h2>
          </div>
          <button type="button" class="rounded-lg border border-red-500/60 px-3 py-2 text-sm font-semibold text-red-300 hover:bg-red-500/10 transition-colors" @click="abrirConfirmacaoLimpezaPagina('simulador')">
            Limpar página
          </button>
        </div>

        <div class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-400 mb-1">Aporte Inicial (R$)</label>
          <input 
            v-model.number="form.principal" 
            type="number" 
            class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-400 mb-1">Aporte Mensal (R$)</label>
          <input 
            v-model.number="form.aporteMensal" 
            type="number" 
            class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
          />
        </div>

        <div class="grid grid-cols-2 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-400 mb-1">Taxa de Juros (% a.m.)</label>
            <input 
              v-model.number="form.taxa" 
              type="number" 
              step="0.01"
              class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-400 mb-1">Período (meses)</label>
            <input 
              v-model.number="form.periodo" 
              type="number" 
              readonly
              class="w-full bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-white focus:outline-none focus:border-emerald-500 transition-colors"
            />
          </div>
        </div>

        <button 
          @click="calcular" 
          class="w-full bg-emerald-500 hover:bg-emerald-600 text-gray-900 font-bold py-3 px-4 rounded-lg transition-colors mt-2"
        >
          Calcular Evolução
        </button>
        </div>

        <div v-if="resultado" class="mt-8 pt-6 border-t border-gray-700 space-y-3">
        <h3 class="text-md font-semibold text-gray-300">Resumo da Simulação:</h3>
        
        <div class="flex justify-between items-center bg-gray-900/50 p-3 rounded-lg">
          <span class="text-sm text-gray-400">Total Investido:</span>
          <span class="font-mono text-white">{{ resultado.totalInvestido }}</span>
        </div>
        
        <div class="flex justify-between items-center bg-gray-900/50 p-3 rounded-lg">
          <span class="text-sm text-gray-400">Total em Juros:</span>
          <span class="font-mono text-emerald-400 font-semibold">{{ resultado.totalJuros }}</span>
        </div>
        
        <div class="flex justify-between items-center bg-emerald-950/30 p-3 rounded-lg border border-emerald-500/20">
          <span class="text-sm font-medium text-emerald-300">Montante Final:</span>
          <span class="font-mono text-xl font-bold text-emerald-400">{{ resultado.montanteFinal }}</span>
        </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import currency from 'currency.js'
import MonthPicker from './components/MonthPicker.vue'

const projeto = reactive({
  nome: '',
  tempoMeses: '',
  inicioPrevisto: '',
  temGarantia: false,
  mesesGarantia: ''
})

const adicionarMeses = (data, meses) => {
  if (!data || !meses) return ''

  const [ano, mes, dia] = data.split('-').map(Number)
  const resultado = new Date(ano, mes - 1 + meses, dia)
  const anoResultado = resultado.getFullYear()
  const mesResultado = String(resultado.getMonth() + 1).padStart(2, '0')
  const diaResultado = String(resultado.getDate()).padStart(2, '0')

  return `${diaResultado}/${mesResultado}/${anoResultado}`
}

const dataTerminoDesenvolvimento = computed(() => adicionarMeses(projeto.inicioPrevisto, projeto.tempoMeses))
const dataGarantiaEncerramento = computed(() => {
  if (!projeto.temGarantia || !projeto.inicioPrevisto || !projeto.tempoMeses) return ''

  const [ano, mes, dia] = projeto.inicioPrevisto.split('-').map(Number)
  const termino = new Date(ano, mes - 1 + projeto.tempoMeses, dia)
  const dataTermino = `${termino.getFullYear()}-${String(termino.getMonth() + 1).padStart(2, '0')}-${String(termino.getDate()).padStart(2, '0')}`

  return adicionarMeses(dataTermino, projeto.mesesGarantia)
})

const totalMesesProjeto = computed(() => {
  const desenvolvimento = Number(projeto.tempoMeses) || 0
  const garantia = projeto.temGarantia ? (Number(projeto.mesesGarantia) || 0) : 0

  return desenvolvimento + garantia
})

const mesInicialProjeto = computed(() => projeto.inicioPrevisto ? projeto.inicioPrevisto.slice(0, 7) : '')
const mesFinalDesenvolvimento = computed(() => {
  if (!mesInicialProjeto.value || !mesesDesenvolvimento.value) return ''

  return adicionarMesesAoMes(mesInicialProjeto.value, mesesDesenvolvimento.value - 1)
})
const mesFinalProjeto = computed(() => {
  if (!mesInicialProjeto.value || !totalMesesProjeto.value) return ''

  return adicionarMesesAoMes(mesInicialProjeto.value, totalMesesProjeto.value - 1)
})

const adicionarMesesAoMes = (mes, quantidade) => {
  if (!mes || !quantidade && quantidade !== 0) return ''

  const [ano, numeroMes] = mes.split('-').map(Number)
  const resultado = new Date(ano, numeroMes - 1 + quantidade, 1)

  return `${resultado.getFullYear()}-${String(resultado.getMonth() + 1).padStart(2, '0')}`
}

const mesesDesenvolvimento = computed(() => Number(projeto.tempoMeses) || 0)

const mesMaximoInicioParticipacao = (membro) => {
  if (!mesFinalProjeto.value || !membro.mesesParticipacao) return mesInicialProjeto.value

  return adicionarMesesAoMes(mesFinalProjeto.value, -(Number(membro.mesesParticipacao) - 1))
}

const calcularFimParticipacao = (membro) => {
  const inicio = membro.inicioParticipacao || mesInicialProjeto.value
  const meses = Number(membro.mesesParticipacao) || 1

  return adicionarMesesAoMes(inicio, meses - 1)
}

const formularioValido = computed(() => {
  const dadosBasicosPreenchidos = projeto.tempoMeses > 0 && projeto.inicioPrevisto
  const garantiaPreenchida = !projeto.temGarantia || projeto.mesesGarantia > 0

  return Boolean(dadosBasicosPreenchidos && garantiaPreenchida)
})

const projetoConfigurado = ref(false)
const paginaAtual = ref('projeto')
const etapas = [
  { id: 'projeto', nome: 'Projeto' },
  { id: 'equipe', nome: 'Equipe' },
  { id: 'alunos', nome: 'Alunos' },
  { id: 'servicos', nome: 'Serviços e locomoção' },
  { id: 'contrapartida', nome: 'Contrapartida' },
  { id: 'simulador', nome: 'Simulação' }
]

const etapaValida = (id) => {
  if (id === 'projeto') return formularioValido.value
  if (id === 'equipe') return equipePreenchida.value
  if (id === 'contrapartida') return equipePreenchida.value
  if (id === 'alunos') return alunosPreenchidos.value
  if (id === 'servicos') return servicosPreenchidos.value
  if (id === 'simulador') return projetoConfigurado.value && equipePreenchida.value && alunosPreenchidos.value && servicosPreenchidos.value

  return false
}

const etapaDisponivel = (id) => {
  const indexSelecionado = etapas.findIndex((etapa) => etapa.id === id)
  if (indexSelecionado <= 0) return true

  return etapas
    .slice(0, indexSelecionado)
    .every((etapa) => etapaValida(etapa.id))
}

const abrirEtapa = (id) => {
  const indexDestino = etapas.findIndex((etapa) => etapa.id === id)
  const indexAtual = etapas.findIndex((etapa) => etapa.id === paginaAtual.value)

  if (indexDestino <= indexAtual || etapaDisponivel(id)) {
    paginaAtual.value = id
  }
}

const equipeFinanceira = reactive([{
  titulacao: '',
  cpf: '',
  nome: '',
  salario: '',
  mesesParticipacao: '',
  inicioParticipacao: '',
  horasSemanais: 10,
  manualAdicao: false
}])

const contrapartidaManual = reactive([])
const linhasContrapartida = computed(() => [...equipeFinanceira, ...contrapartidaManual])

const proximoIdAluno = ref(2)
const alunos = reactive([{
  id: 1,
  titulacao: '',
  nome: 'Aluno 1',
  mesesParticipacao: '',
  inicioParticipacao: '',
  horasSemanais: 20,
  valorBolsa: ''
}])

const proximoIdServico = ref(2)
const servicosLocomocao = reactive([{
  id: 1,
  tipo: '',
  descricao: '',
  unidade: 1,
  valor: ''
}])
const mensagemImportacaoCadastro = ref('')
const mensagemImportacaoCsv = ref('')
const confirmacaoLimpezaAberta = ref(false)
const confirmacaoLimpezaPagina = ref('')
const nomePaginaLimpeza = computed(() => ({
  projeto: 'o projeto',
  'dados-projeto': 'os dados do projeto',
  equipe: 'a equipe financeira',
  alunos: 'os alunos',
  servicos: 'os serviços e a locomoção',
  simulador: 'a simulação'
}[confirmacaoLimpezaPagina.value] || 'esta página'))

const abrirConfirmacaoLimpezaPagina = (pagina) => {
  confirmacaoLimpezaPagina.value = pagina
}

const limparPaginaLocal = () => {
  const pagina = confirmacaoLimpezaPagina.value

  if (pagina === 'projeto' || pagina === 'dados-projeto') {
    Object.assign(projeto, {
      nome: '',
      tempoMeses: '',
      inicioPrevisto: '',
      temGarantia: false,
      mesesGarantia: ''
    })
    form.periodo = ''
    projetoConfigurado.value = false
    paginaAtual.value = 'projeto'
  }

  if (pagina === 'equipe') {
    equipeFinanceira.splice(0, equipeFinanceira.length, {
      titulacao: '',
      cpf: '',
      nome: '',
      salario: '',
      mesesParticipacao: '',
      inicioParticipacao: '',
      horasSemanais: 10,
      erroMesesParticipacao: ''
    })
  }

  if (pagina === 'alunos') {
    alunos.splice(0, alunos.length, {
      id: 1,
      titulacao: '',
      nome: 'Aluno 1',
      mesesParticipacao: '',
      inicioParticipacao: '',
      horasSemanais: 20,
      valorBolsa: '',
      erroMesesParticipacao: ''
    })
    proximoIdAluno.value = 2
  }

  if (pagina === 'servicos') {
    servicosLocomocao.splice(0, servicosLocomocao.length, {
      id: 1,
      tipo: '',
      descricao: '',
      unidade: 1,
      valor: ''
    })
    proximoIdServico.value = 2
  }

  if (pagina === 'simulador') {
    form.principal = ''
    form.aporteMensal = ''
    form.taxa = ''
    form.periodo = projeto.tempoMeses || ''
    resultado.value = null
  }

  confirmacaoLimpezaPagina.value = ''
}

const abrirConfirmacaoLimpeza = () => {
  confirmacaoLimpezaAberta.value = true
}

const limparFormularios = () => {
  Object.assign(projeto, {
    nome: '',
    tempoMeses: '',
    inicioPrevisto: '',
    temGarantia: false,
    mesesGarantia: ''
  })

  Object.assign(equipeFinanceira[0], {
    titulacao: '',
    cpf: '',
    nome: '',
    salario: '',
    mesesParticipacao: '',
    inicioParticipacao: '',
    horasSemanais: 10,
    erroMesesParticipacao: ''
  })
  equipeFinanceira.splice(1)

  Object.assign(alunos[0], {
    id: 1,
    titulacao: '',
    nome: 'Aluno 1',
    mesesParticipacao: '',
    inicioParticipacao: '',
    horasSemanais: 20,
    valorBolsa: '',
    erroMesesParticipacao: ''
  })
  alunos.splice(1)

  servicosLocomocao.splice(0, servicosLocomocao.length, {
    id: 1,
    tipo: '',
    descricao: '',
    unidade: 1,
    valor: ''
  })

  form.principal = ''
  form.aporteMensal = ''
  form.taxa = ''
  form.periodo = ''
  resultado.value = null
  proximoIdAluno.value = 2
  proximoIdServico.value = 2
  projetoConfigurado.value = false
  paginaAtual.value = 'projeto'
  confirmacaoLimpezaAberta.value = false
}

const formatarCpfTexto = (cpf) => {
  const digitos = String(cpf || '').replace(/\D/g, '').slice(0, 11)
  return digitos
    .replace(/(\d{3})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d{1,2})$/, '$1-$2')
}

const importarCadastroCsv = (evento) => {
  const arquivo = evento.target.files?.[0]
  evento.target.value = ''
  mensagemImportacaoCadastro.value = ''

  if (!arquivo) return

  const leitor = new FileReader()
  leitor.onload = () => {
    const linhas = String(leitor.result || '')
      .replace(/^\uFEFF/, '')
      .split(/\r?\n/)
      .filter((linha) => linha.trim())

    if (linhas.length < 2) {
      mensagemImportacaoCadastro.value = 'O CSV precisa ter cabeçalho e registros.'
      return
    }

    const separador = (linhas[0].match(/;/g) || []).length > (linhas[0].match(/,/g) || []).length ? ';' : ','
    const cabecalho = dividirLinhaCsv(linhas[0], separador).map(normalizarTextoCsv)
    const indice = (nome) => cabecalho.indexOf(nome)
    const colunasObrigatorias = ['registro', 'nome', 'titulacao', 'cpf', 'salario', 'horas semanais', 'meses participacao', 'valor bolsa']

    if (colunasObrigatorias.some((coluna) => indice(coluna) < 0)) {
      mensagemImportacaoCadastro.value = 'Use as colunas do modelo CSV baixado nesta tela.'
      return
    }

    const pesquisadores = []
    const alunosImportados = []
    const servicosImportados = []
    let projetoImportado = null

    linhas.slice(1).forEach((linha) => {
      const colunas = dividirLinhaCsv(linha, separador)
      const registro = normalizarTextoCsv(colunas[indice('registro')] || '')
      const nome = (colunas[indice('nome')] || '').trim()
      const titulacao = (colunas[indice('titulacao')] || '').trim()
      const meses = converterNumeroCsv(colunas[indice('meses participacao')])
      const horas = converterNumeroCsv(colunas[indice('horas semanais')])

      if (registro === 'projeto') {
        const temGarantia = normalizarTextoCsv(colunas[indice('tem garantia')] || '')
        projetoImportado = {
          nome,
          tempoMeses: converterNumeroCsv(colunas[indice('tempo desenvolvimento')]),
          inicioPrevisto: (colunas[indice('inicio projeto')] || '').trim(),
          temGarantia: ['sim', 'true', '1', 'yes'].includes(temGarantia),
          mesesGarantia: converterNumeroCsv(colunas[indice('meses garantia')])
        }
        return
      }

      if (registro === 'servico') {
        const tipoTexto = normalizarTextoCsv(colunas[indice('tipo')] || '')
        const item = {
          id: servicosImportados.length + 1,
          tipo: tiposServico[tipoTexto] || tipoTexto,
          descricao: (colunas[indice('descricao')] || '').trim(),
          unidade: converterNumeroCsv(colunas[indice('unidade')]),
          valor: converterNumeroCsv(colunas[indice('valor')])
        }
        if (linhaServicoPreenchida(item)) servicosImportados.push(item)
        return
      }

      if (!nome || !titulacao || !Number.isFinite(meses) || meses < 1) return

      if (registro === 'pesquisador' || registro === 'equipe') {
        pesquisadores.push({
          titulacao,
          cpf: formatarCpfTexto(colunas[indice('cpf')]),
          nome,
          salario: converterNumeroCsv(colunas[indice('salario')]),
          mesesParticipacao: meses,
          inicioParticipacao: (colunas[indice('inicio participacao')] || '').trim(),
          horasSemanais: Number.isFinite(horas) && horas > 0 ? horas : 10,
          manualAdicao: false
        })
      }

      if (registro === 'aluno') {
        alunosImportados.push({
          id: alunosImportados.length + 1,
          titulacao,
          nome,
          mesesParticipacao: meses,
          inicioParticipacao: (colunas[indice('inicio participacao')] || '').trim(),
          horasSemanais: Number.isFinite(horas) && horas > 0 ? horas : 20,
          valorBolsa: converterNumeroCsv(colunas[indice('valor bolsa')])
        })
      }
    })

    if (!projetoImportado && !pesquisadores.length && !alunosImportados.length && !servicosImportados.length) {
      mensagemImportacaoCadastro.value = 'Nenhum dado válido foi encontrado no CSV.'
      return
    }

    if (projetoImportado) {
      Object.assign(projeto, projetoImportado)
      form.periodo = projeto.tempoMeses
    }
    if (pesquisadores.length) equipeFinanceira.splice(0, equipeFinanceira.length, ...pesquisadores)
    if (alunosImportados.length) {
      alunos.splice(0, alunos.length, ...alunosImportados)
      proximoIdAluno.value = alunosImportados.length + 1
    }
    if (servicosImportados.length) servicosLocomocao.splice(0, servicosLocomocao.length, ...servicosImportados)

    projetoConfigurado.value = false
    paginaAtual.value = 'projeto'
    resultado.value = null

    mensagemImportacaoCadastro.value = `${pesquisadores.length} pesquisador(es), ${alunosImportados.length} aluno(s) e ${servicosImportados.length} serviço(s) importado(s). Revise o projeto e avance.`
  }

  leitor.onerror = () => {
    mensagemImportacaoCadastro.value = 'Não foi possível ler o arquivo CSV.'
  }
  leitor.readAsText(arquivo, 'UTF-8')
}

const baixarEPreencherModeloCadastro = () => {
  Object.assign(projeto, {
    nome: 'Sistema de inovação',
    tempoMeses: 24,
    inicioPrevisto: '2026-10-01',
    temGarantia: false,
    mesesGarantia: ''
  })

  const pesquisadores = [
    ['Mestrado', '529.982.247-25', 'Ana Souza', 8500, 20],
    ['Doutorado', '111.444.777-35', 'Bruno Lima', 10500, 20],
    ['Especialização', '935.411.347-80', 'Carla Mendes', 7200, 15],
    ['Graduação', '153.509.460-56', 'Daniel Alves', 5600, 10]
  ].map(([titulacao, cpf, nome, salario, horasSemanais]) => ({
    titulacao,
    cpf,
    nome,
    salario,
    mesesParticipacao: 24,
    inicioParticipacao: '2026-10',
    horasSemanais,
    manualAdicao: false
  }))

  const alunosModelo = [
    ['Eduardo Santos', 'Graduação', 1800],
    ['Fernanda Costa', 'Graduação', 1800],
    ['Gabriel Oliveira', 'Técnico', 1500],
    ['Helena Martins', 'Graduação', 1800],
    ['Igor Rodrigues', 'Mestrado', 2200]
  ].map(([nome, titulacao, valorBolsa], indice) => ({
    id: indice + 1,
    titulacao,
    nome,
    mesesParticipacao: 22,
    inicioParticipacao: '2026-12',
    horasSemanais: 20,
    valorBolsa
  }))

  equipeFinanceira.splice(0, equipeFinanceira.length, ...pesquisadores)
  alunos.splice(0, alunos.length, ...alunosModelo)
  proximoIdAluno.value = alunosModelo.length + 1

  servicosLocomocao.splice(0, servicosLocomocao.length, {
    id: 1,
    tipo: 'diarias',
    descricao: 'Viagem para reunião técnica',
    unidade: 2,
    valor: 450
  })
  projetoConfigurado.value = false
  paginaAtual.value = 'projeto'
  form.periodo = projeto.tempoMeses
  resultado.value = null

  const conteudo = 'registro;nome;titulacao;cpf;salario;horas semanais;meses participacao;inicio participacao;valor bolsa;tempo desenvolvimento;inicio projeto;tem garantia;meses garantia;tipo;descricao;unidade;valor\n'
    + 'projeto;Sistema de inovação;;;;;;;;24;2026-10-01;não;0;;;;\n'
    + pesquisadores.map((membro) => `pesquisador;${membro.nome};${membro.titulacao};${membro.cpf};${membro.salario.toFixed(2).replace('.', ',')};${membro.horasSemanais};${membro.mesesParticipacao};${membro.inicioParticipacao};;`).join('\n')
    + '\n'
    + alunosModelo.map((aluno) => `aluno;${aluno.nome};${aluno.titulacao};;;${aluno.horasSemanais};${aluno.mesesParticipacao};${aluno.inicioParticipacao};${aluno.valorBolsa.toFixed(2).replace('.', ',')}`).join('\n')
    + '\nservico;;;;;;;;;;;;;Diárias;Viagem para reunião técnica;2;450,00'
    + '\n'
  const arquivo = new Blob([`\uFEFF${conteudo}`], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(arquivo)
  const link = document.createElement('a')
  link.href = url
  link.download = 'modelo-cadastro-completo.csv'
  link.click()
  URL.revokeObjectURL(url)

  mensagemImportacaoCadastro.value = 'Modelo baixado e dados preenchidos: 4 pesquisadores e 5 alunos.'
}

const tiposServico = {
  diarias: 'diarias',
  'passagens aereas': 'passagens-aereas',
  'servicos de terceiros': 'servicos-terceiros',
  assinaturas: 'assinaturas',
  'compra de bens fisicos': 'bens-fisicos'
}

const normalizarTextoCsv = (valor) => valor
  .trim()
  .toLowerCase()
  .normalize('NFD')
  .replace(/[\u0300-\u036f]/g, '')

const dividirLinhaCsv = (linha, separador) => {
  const valores = []
  let valor = ''
  let entreAspas = false

  for (let indice = 0; indice < linha.length; indice += 1) {
    const caractere = linha[indice]
    const proximo = linha[indice + 1]

    if (caractere === '"' && entreAspas && proximo === '"') {
      valor += '"'
      indice += 1
    } else if (caractere === '"') {
      entreAspas = !entreAspas
    } else if (caractere === separador && !entreAspas) {
      valores.push(valor.trim())
      valor = ''
    } else {
      valor += caractere
    }
  }

  valores.push(valor.trim())
  return valores
}

const converterNumeroCsv = (valor) => {
  const texto = String(valor || '').trim().replace(/R\$\s*/gi, '')
  if (!texto) return NaN

  const numero = texto.includes(',')
    ? texto.replace(/\./g, '').replace(',', '.')
    : texto

  return Number(numero)
}

const importarOrcamentoCsv = (evento) => {
  const arquivo = evento.target.files?.[0]
  evento.target.value = ''
  mensagemImportacaoCsv.value = ''

  if (!arquivo) return

  const leitor = new FileReader()
  leitor.onload = () => {
    const linhas = String(leitor.result || '')
      .replace(/^\uFEFF/, '')
      .split(/\r?\n/)
      .filter((linha) => linha.trim())

    if (linhas.length < 2) {
      mensagemImportacaoCsv.value = 'O CSV precisa ter cabeçalho e pelo menos uma linha.'
      return
    }

    const separador = (linhas[0].match(/;/g) || []).length > (linhas[0].match(/,/g) || []).length ? ';' : ','
    const cabecalho = dividirLinhaCsv(linhas[0], separador).map(normalizarTextoCsv)
    const indiceTipo = cabecalho.indexOf('tipo')
    const indiceDescricao = cabecalho.indexOf('descricao')
    const indiceUnidade = cabecalho.indexOf('unidade')
    const indiceValor = cabecalho.indexOf('valor')

    if ([indiceTipo, indiceDescricao, indiceUnidade, indiceValor].some((indice) => indice < 0)) {
      mensagemImportacaoCsv.value = 'Use as colunas: tipo, descricao, unidade e valor.'
      return
    }

    const itens = linhas.slice(1).map((linha) => {
      const colunas = dividirLinhaCsv(linha, separador)
      const tipoTexto = normalizarTextoCsv(colunas[indiceTipo] || '')
      const item = {
        id: 0,
        tipo: tiposServico[tipoTexto] || '',
        descricao: (colunas[indiceDescricao] || '').trim(),
        unidade: converterNumeroCsv(colunas[indiceUnidade]),
        valor: converterNumeroCsv(colunas[indiceValor])
      }

      return item
    }).filter(linhaServicoPreenchida)

    if (!itens.length) {
      mensagemImportacaoCsv.value = 'Nenhuma linha válida foi encontrada no CSV.'
      return
    }

    servicosLocomocao.splice(0, servicosLocomocao.length, ...itens.map((item, indice) => ({
      ...item,
      id: indice + 1
    })))
    proximoIdServico.value = itens.length + 1
    mensagemImportacaoCsv.value = `${itens.length} item(ns) importado(s) com sucesso.`
  }

  leitor.onerror = () => {
    mensagemImportacaoCsv.value = 'Não foi possível ler o arquivo CSV.'
  }
  leitor.readAsText(arquivo, 'UTF-8')
}

const baixarModeloCsv = () => {
  const conteudo = 'tipo;descricao;unidade;valor\nDiárias;Exemplo de diária;1;150,00\n'
  const arquivo = new Blob([`\uFEFF${conteudo}`], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(arquivo)
  const link = document.createElement('a')
  link.href = url
  link.download = 'modelo-orcamento.csv'
  link.click()
  URL.revokeObjectURL(url)
}

const cpfValido = (cpf) => {
  const digitos = cpf.replace(/\D/g, '')

  if (digitos.length !== 11 || /^([0-9])\1+$/.test(digitos)) return false

  let soma = 0
  for (let indice = 0; indice < 9; indice++) {
    soma += Number(digitos[indice]) * (10 - indice)
  }

  let resto = (soma * 10) % 11
  if (resto === 10) resto = 0
  if (resto !== Number(digitos[9])) return false

  soma = 0
  for (let indice = 0; indice < 10; indice++) {
    soma += Number(digitos[indice]) * (11 - indice)
  }

  resto = (soma * 10) % 11
  if (resto === 10) resto = 0

  return resto === Number(digitos[10])
}

const formatarCpf = (membro, evento) => {
  const digitos = evento.target.value.replace(/\D/g, '').slice(0, 11)
  membro.cpf = digitos
    .replace(/(\d{3})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d)/, '$1.$2')
    .replace(/(\d{3})(\d{1,2})$/, '$1-$2')
}

const formatarMoeda = (valor) => {
  const numero = Number(valor)

  if (!Number.isFinite(numero)) return '—'

  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL'
  }).format(numero)
}

const atualizarNomeContrapartida = (membro, evento) => {
  if (!membro.manualAdicao) return
  membro.nome = evento.target.value
}

const atualizarSalarioContrapartida = (membro, evento) => {
  if (!membro.manualAdicao) return
  membro.salario = evento.target.value
}

const obterValorHora = (salario, naoDocente = false) => {
  const valor = Number(salario)
  if (!Number.isFinite(valor) || valor <= 0) return '—'

  let fator = 0

  if (naoDocente) {
    fator = ((valor * 13) / 12) / 160
  } else {
    fator = ((valor * 13.5) / 12) / 160
  }

  return formatarMoeda(fator)
}

const obterHorasSemanais = (horasMensais) => {
  const valor = Number(horasMensais)
  if (!Number.isFinite(valor) || valor <= 0) return '—'

  const horasSemanais = valor / 4
  return `${Number.isInteger(horasSemanais) ? horasSemanais : horasSemanais.toFixed(1)}h/semana`
}

const mesParaExibicao = (mes) => {
  if (!mes || !/^\d{4}-\d{2}$/.test(mes)) return ''

  const [ano, numeroMes] = mes.split('-')
  return `${numeroMes}/${ano}`
}

const formatarMesAno = (registro, evento) => {
  const numeros = evento.target.value.replace(/\D/g, '').slice(0, 6)
  registro.inicioParticipacao = numeros.length > 2
    ? `${numeros.slice(2)}-${numeros.slice(0, 2)}`
    : numeros
}

const formatarMesAnoEquipe = (membro, evento) => {
  const numeros = evento.target.value.replace(/\D/g, '').slice(0, 6)
  const valor = numeros.length > 2
    ? `${numeros.slice(2)}-${numeros.slice(0, 2)}`
    : numeros

  const inicioMinimo = mesInicialProjeto.value
  const inicioMaximo = mesMaximoInicioParticipacao(membro)

  if (!valor) {
    membro.inicioParticipacao = ''
    return
  }

  if (!mesAnoValido(valor, mesInicialProjeto.value, mesFinalProjeto.value)) {
    membro.inicioParticipacao = inicioMaximo
    return
  }

  membro.inicioParticipacao = valor < inicioMinimo
    ? inicioMinimo
    : valor > inicioMaximo
      ? inicioMaximo
      : valor
}

const formatarMesAnoAluno = (aluno, evento) => {
  const numeros = evento.target.value.replace(/\D/g, '').slice(0, 6)
  const valor = numeros.length > 2
    ? `${numeros.slice(2)}-${numeros.slice(0, 2)}`
    : numeros

  const inicioMinimo = mesInicialProjeto.value
  const inicioMaximo = mesMaximoInicioParticipacao({ mesesParticipacao: aluno.mesesParticipacao })

  if (!valor) {
    aluno.inicioParticipacao = ''
    return
  }

  if (!mesAnoValido(valor, mesInicialProjeto.value, mesFinalProjeto.value)) {
    aluno.inicioParticipacao = inicioMaximo
    return
  }

  aluno.inicioParticipacao = valor < inicioMinimo
    ? inicioMinimo
    : valor > inicioMaximo
      ? inicioMaximo
      : valor
}

const atualizarInicioEquipe = (membro, valor) => {
  const inicioMaximo = mesMaximoInicioParticipacao(membro)

  if (!valor) {
    membro.inicioParticipacao = ''
    membro.erroInicioParticipacao = 'Informe o início da participação.'
    return
  }

  if (mesAnoValido(valor, mesInicialProjeto.value, inicioMaximo)) {
    membro.inicioParticipacao = valor
    membro.erroInicioParticipacao = ''
    return
  }

  membro.inicioParticipacao = inicioMaximo
  membro.erroInicioParticipacao = 'O início informado ultrapassa o limite de meses permitido para este membro.'
}

const atualizarInicioAluno = (aluno, valor) => {
  const inicioMaximo = mesMaximoInicioParticipacao({ mesesParticipacao: aluno.mesesParticipacao })
  if (mesAnoValido(valor, mesInicialProjeto.value, inicioMaximo)) {
    aluno.inicioParticipacao = valor
  }
}

const mesAnoValido = (mes, minimo, maximo) => (
  /^\d{4}-(0[1-9]|1[0-2])$/.test(mes) &&
  mes >= minimo &&
  mes <= maximo
)

const calcularInicioPadraoEquipe = (meses) => {
  if (!mesInicialProjeto.value || !meses) return ''

  return mesInicialProjeto.value
}

const temporizadoresErroMeses = new WeakMap()

const informarErroMeses = (registro, mensagem) => {
  const temporizadorAnterior = temporizadoresErroMeses.get(registro)
  if (temporizadorAnterior) clearTimeout(temporizadorAnterior)

  registro.erroMesesParticipacao = mensagem
  const temporizador = setTimeout(() => {
    registro.erroMesesParticipacao = ''
    temporizadoresErroMeses.delete(registro)
  }, 3000)
  temporizadoresErroMeses.set(registro, temporizador)
}

const limitarMesesParticipacao = (membro, evento) => {
  if (evento.target.value === '') {
    membro.mesesParticipacao = ''
    membro.inicioParticipacao = ''
    membro.erroInicioParticipacao = 'Informe o início da participação.'
    informarErroMeses(membro, 'Informe uma quantidade de meses maior que zero.')
    return
  }

  const meses = Number(evento.target.value)
  if (meses > mesesDesenvolvimento.value) {
    informarErroMeses(membro, `O máximo permitido é ${mesesDesenvolvimento.value} meses.`)
  } else if (meses < 1) {
    informarErroMeses(membro, 'Informe uma quantidade de meses maior que zero.')
  }
  membro.mesesParticipacao = Math.min(Math.max(meses, 1), mesesDesenvolvimento.value)

  if (!membro.inicioParticipacao || !mesAnoValido(membro.inicioParticipacao, mesInicialProjeto.value, mesFinalProjeto.value)) {
    membro.inicioParticipacao = mesInicialProjeto.value
    membro.erroInicioParticipacao = ''
    return
  }

  const inicioMaximo = mesMaximoInicioParticipacao(membro)
  if (membro.inicioParticipacao > inicioMaximo) {
    membro.inicioParticipacao = inicioMaximo
    membro.erroInicioParticipacao = 'O início informado ultrapassa o limite de meses permitido para este membro.'
    return
  }

  membro.erroInicioParticipacao = ''
}

const linhaEquipePreenchida = (membro) => {
  const inicioDefinitivo = membro.inicioParticipacao || mesInicialProjeto.value
  const fimDefinitivo = calcularFimParticipacao(membro)
  const inicioMaximo = mesMaximoInicioParticipacao(membro)

  return (
    membro.titulacao.trim() &&
    cpfValido(membro.cpf) &&
    membro.nome.trim() &&
    membro.salario !== '' &&
    membro.mesesParticipacao >= 1 &&
    membro.mesesParticipacao <= mesesDesenvolvimento.value &&
    inicioDefinitivo &&
    mesAnoValido(inicioDefinitivo, mesInicialProjeto.value, mesFinalProjeto.value) &&
    inicioDefinitivo >= mesInicialProjeto.value &&
    inicioDefinitivo <= inicioMaximo &&
    fimDefinitivo &&
    mesAnoValido(fimDefinitivo, mesInicialProjeto.value, mesFinalProjeto.value) &&
    fimDefinitivo <= mesFinalProjeto.value &&
    membro.horasSemanais >= 1
  )
}

const maximoMesesAluno = computed(() => Math.max(Number(projeto.tempoMeses) - 2, 0))

const calcularInicioPadraoAluno = (meses) => {
  if (!mesFinalProjeto.value || !meses) return ''

  return mesMaximoInicioParticipacao({ mesesParticipacao: meses })
}

const calcularFimAluno = (aluno) => {
  const inicio = aluno.inicioParticipacao || calcularInicioPadraoAluno(aluno.mesesParticipacao)
  if (!inicio || !aluno.mesesParticipacao) return ''

  return adicionarMesesAoMes(inicio, Number(aluno.mesesParticipacao) - 1)
}

const limitarMesesAluno = (aluno, evento) => {
  if (evento.target.value === '') {
    aluno.mesesParticipacao = ''
    aluno.inicioParticipacao = ''
    informarErroMeses(aluno, 'Informe uma quantidade de meses maior que zero.')
    return
  }

  const meses = Number(evento.target.value)
  if (meses > maximoMesesAluno.value) {
    informarErroMeses(aluno, `O máximo permitido é ${maximoMesesAluno.value} meses.`)
  } else if (meses < 1) {
    informarErroMeses(aluno, 'Informe uma quantidade de meses maior que zero.')
  }
  aluno.mesesParticipacao = Math.min(
    Math.max(meses, 1),
    maximoMesesAluno.value
  )

  if (!aluno.inicioParticipacao || !mesAnoValido(aluno.inicioParticipacao, mesInicialProjeto.value, mesFinalProjeto.value)) {
    aluno.inicioParticipacao = calcularInicioPadraoAluno(aluno.mesesParticipacao)
  }
}

const linhaAlunoPreenchida = (aluno) => {
  const inicioDefinitivo = aluno.inicioParticipacao || calcularInicioPadraoAluno(aluno.mesesParticipacao)
  const fimDefinitivo = calcularFimAluno(aluno)
  const inicioMaximo = mesMaximoInicioParticipacao({ mesesParticipacao: aluno.mesesParticipacao })

  return (
    aluno.titulacao.trim() &&
    aluno.nome.trim() &&
    aluno.mesesParticipacao >= 1 &&
    aluno.mesesParticipacao <= maximoMesesAluno.value &&
    inicioDefinitivo &&
    mesAnoValido(inicioDefinitivo, mesInicialProjeto.value, mesFinalProjeto.value) &&
    inicioDefinitivo >= mesInicialProjeto.value &&
    inicioDefinitivo <= inicioMaximo &&
    fimDefinitivo &&
    mesAnoValido(fimDefinitivo, mesInicialProjeto.value, mesFinalProjeto.value) &&
    fimDefinitivo <= mesFinalProjeto.value &&
    aluno.horasSemanais >= 1
  )
}

const alunosPreenchidos = computed(() => (
  maximoMesesAluno.value >= 1 && alunos.length > 0 && alunos.every(linhaAlunoPreenchida)
))

const equipePreenchida = computed(() => (
  equipeFinanceira.length > 0 && equipeFinanceira.every(linhaEquipePreenchida)
))

const adicionarMembro = () => {
  const mesesPadrao = mesesDesenvolvimento.value

  equipeFinanceira.push({
    titulacao: '',
    cpf: '',
    nome: '',
    salario: '',
    mesesParticipacao: mesesPadrao,
    inicioParticipacao: calcularInicioPadraoEquipe(mesesPadrao),
    horasSemanais: 40,
    naoDocente: false,
    manualAdicao: true,
    erroMesesParticipacao: '',
    erroInicioParticipacao: ''
  })
}

const adicionarMembroContrapartida = () => {
  const mesesPadrao = mesesDesenvolvimento.value

  contrapartidaManual.push({
    titulacao: '',
    cpf: '',
    nome: '',
    salario: '',
    mesesParticipacao: mesesPadrao,
    inicioParticipacao: calcularInicioPadraoEquipe(mesesPadrao),
    horasSemanais: 40,
    naoDocente: false,
    manualAdicao: true,
    erroMesesParticipacao: '',
    erroInicioParticipacao: ''
  })
}

const adicionarAluno = () => {
  const id = proximoIdAluno.value
  const mesesPadrao = maximoMesesAluno.value

  alunos.push({
    id,
    titulacao: '',
    nome: `Aluno ${id}`,
    mesesParticipacao: mesesPadrao,
    inicioParticipacao: calcularInicioPadraoAluno(mesesPadrao),
    horasSemanais: 20,
    valorBolsa: ''
  })
  proximoIdAluno.value += 1
}

const linhaServicoPreenchida = (item) => (
  Boolean(item.tipo && item.descricao.trim() && item.unidade >= 1 && item.valor !== '' && Number(item.valor) >= 0)
)

const servicosPreenchidos = computed(() => (
  servicosLocomocao.length > 0 && servicosLocomocao.every(linhaServicoPreenchida)
))

const adicionarServico = () => {
  servicosLocomocao.push({
    id: proximoIdServico.value,
    tipo: '',
    descricao: '',
    unidade: 1,
    valor: ''
  })
  proximoIdServico.value += 1
}

const removerServico = (indice) => {
  if (servicosLocomocao.length === 1) {
    Object.assign(servicosLocomocao[0], {
      tipo: '',
      descricao: '',
      unidade: 1,
      valor: ''
    })
    return
  }

  servicosLocomocao.splice(indice, 1)
}

const removerAluno = (indice) => {
  if (alunos.length === 1) {
    Object.assign(alunos[0], {
      titulacao: '',
      nome: 'Aluno 1',
      mesesParticipacao: maximoMesesAluno.value,
      inicioParticipacao: '',
      horasSemanais: 20,
      valorBolsa: ''
    })
    return
  }

  alunos.splice(indice, 1)
}

const removerMembro = (indice) => {
  if (equipeFinanceira.length === 1) {
    Object.assign(equipeFinanceira[0], {
      titulacao: '',
      cpf: '',
      nome: '',
      salario: '',
      mesesParticipacao: projeto.tempoMeses,
      inicioParticipacao: '',
      horasSemanais: 10
    })
    return
  }

  equipeFinanceira.splice(indice, 1)
}

const configurarProjeto = () => {
  form.periodo = projeto.tempoMeses

  equipeFinanceira.forEach((membro) => {
    membro.mesesParticipacao = Math.min(
      Math.max(Number(membro.mesesParticipacao) || mesesDesenvolvimento.value, 1),
      mesesDesenvolvimento.value
    )
    if (!membro.inicioParticipacao) {
      membro.inicioParticipacao = calcularInicioPadraoEquipe(membro.mesesParticipacao)
    }
  })

  alunos.forEach((aluno) => {
    aluno.mesesParticipacao = Math.min(
      Math.max(Number(aluno.mesesParticipacao) || maximoMesesAluno.value, 1),
      maximoMesesAluno.value
    )
    if (!aluno.inicioParticipacao) {
      aluno.inicioParticipacao = calcularInicioPadraoAluno(aluno.mesesParticipacao)
    }
  })

  projetoConfigurado.value = true
  paginaAtual.value = 'equipe'
}

// Estado dos inputs do formulário
const form = reactive({
  principal: '',
  aporteMensal: '',
  taxa: '',
  periodo: ''
})

// Estado para guardar os resultados do cálculo
const resultado = ref(null)

// Configuração padrão do currency.js para o Real Brasileiro (R$)
const BRL = (valor) => currency(valor, { symbol: 'R$ ', separator: '.', decimal: ',', precision: 2 })

const calcular = () => {
  let totalAcumulado = BRL(form.principal)
  let totalInvestido = BRL(form.principal)
  
  const taxaMensal = form.taxa / 100

  for (let i = 1; i <= form.periodo; i++) {
    // Aplica os juros sobre o montante atual
    const jurosDoMes = totalAcumulado.multiply(taxaMensal)
    totalAcumulado = totalAcumulado.add(jurosDoMes)
    
    // Se não for o primeiro mês, adiciona o aporte mensal
    if (i > 0 && form.aporteMensal > 0) {
      totalAcumulado = totalAcumulado.add(form.aporteMensal)
      totalInvestido = totalInvestido.add(form.aporteMensal)
    }
  }

  const totalJuros = totalAcumulado.subtract(totalInvestido)

  // Atualiza a tela com os valores formatados em R$
  resultado.value = {
    totalInvestido: totalInvestido.format(),
    totalJuros: totalJuros.format(),
    montanteFinal: totalAcumulado.format()
  }
}
</script>
