<template>
    <section class="form">
        <form>
            <span v-if="disciplinas">
                <div class="field">
                    <label for="disciplinas">Disciplina</label>
                    <select id="disciplinas" v-model="disciplina" @change="obterAnotacoes()">
                        <option v-for="disciplina in disciplinas" :value="disciplina">{{ disciplina.disciplina }}</option>
                    </select>
                </div>
                <div class="field">
                    <label for="mes">Mês</label>
                    <input type="month" id="mes" v-model="mes" @change="obterAnotacoes()" />
                </div>

                <div class="field no-print" v-if="anotacoes?.length ?? 0 >= 1">
                    <span>
                        <input type="checkbox" id="titulos" name="titulos" v-model="exibirTitulos" />
                        <label for="titulos">Exibir títulos</label>
                    </span>
                </div>
            </span>

            <div v-else class="info">
                <mark class="info">Não existem anotações.</mark>
            </div>

            <div v-else class="info">
                <mark class="info">Não existem disciplinas cadastradas.</mark>
            </div>
        </form>
    </section>

    <span v-for="anotacao in anotacoes">
        <AnotacaoComponent :anotacao="anotacao" :disciplina="disciplina" :exibir-titulos="exibirTitulos" origem="anotacao" @go-to-page="goToPage">
        </AnotacaoComponent>
    </span>
</template>

<script lang="ts">
import AuthService from '@/Services/AuthService';
import Anotacao from '@/Models/Anotacao';
import Dia from '@/Models/Dia';
import { defineComponent } from 'vue';
import MarkdownIt from 'markdown-it';
import Disciplina from '@/Models/Disciplina';
import AnotacaoComponent from '@/Pages/Components/Anotacao.vue';
import AnotacaoService from '@/Services/AnotacaoService';
import DisciplinaService from '@/Services/DisciplinaService';

export default defineComponent({
    name: 'AnotacoesView',

    components: {
        AnotacaoComponent
    },

    data(): {
        anotacaoService: AnotacaoService,
        disciplinaService: DisciplinaService,
        disciplinas: Disciplina[] | undefined,
        disciplina: Disciplina | undefined,
        exibirTitulos: boolean,
        anotacoes: Anotacao[] | undefined,
        mes: string | undefined,
        md: MarkdownIt,

    } {
        return {
            anotacaoService: new AnotacaoService(this.axios),
            disciplinaService: new DisciplinaService(this.axios),
            disciplinas: undefined,
            disciplina: undefined,
            exibirTitulos: true,
            anotacoes: undefined,
            mes: undefined,
            md: new MarkdownIt({
                xhtmlOut: true,
                breaks: true,
                linkify: true
            })
        }
    },

    emits: ['goToPage'],

    methods: {
        goToPage(page: string) {
            this.$emit('goToPage', page);
        },
        async obterDisciplinas() {
            try {
                this.disciplinas = await this.disciplinaService.obter();
                if (this.disciplinas) {
                    const id = sessionStorage.getItem('anotacoes_disciplina');
                    const disciplina = this.disciplinas.find(d => d.id == parseInt(id ?? '0'));
                    this.disciplina = disciplina ?? this.disciplinas[0];
                }
                return true;
            } catch (error) {
                return false;
            }
        },
        async obterAnotacoes() {
            if (!this.disciplina || !this.mes) {
                return;
            }

            this.anotacoes = await this.anotacaoService.obterAnotacoes(this.disciplina, this.mes);
        },
        obterTitulo(anotacao: Anotacao) {
            return `Aula ${anotacao.aula} - ${Dia.dataCompleta(new Date(anotacao.data))}.`;
        },
        obterConteudo(anotacao: Anotacao) {
            return this.md.render(anotacao.anotacao!);
        },
        obterDisciplina(id_disciplina: number | undefined): string | undefined {
            return this.disciplinas?.find(d => d.id == id_disciplina)?.disciplina;
        }
    },

    async mounted() {
        if (!AuthService.autenticado) {
            this.goToPage('Home');
            return;
        }

        let exibirTitulos = localStorage.getItem('exibir_titulos');
        if (exibirTitulos) {
            this.exibirTitulos = exibirTitulos == 's';
        }

        this.mes = sessionStorage.getItem('anotacoes_mes') ?? Dia.mesAtual();
        if (await this.obterDisciplinas()) {
            await this.obterAnotacoes();
        }

    },

    watch: {
        exibirTitulos(newData: boolean) {
            localStorage.setItem('exibir_titulos', newData ? 's' : 'n');
        },

        mes(newData: string) {
            sessionStorage.setItem('anotacoes_mes', newData);
        },

        disciplina(newData: Disciplina) {
            sessionStorage.setItem('anotacoes_disciplina', newData.id?.toString() ?? '');
        }
    }
});
</script>

<style scoped>
.anotacoes_label {
    display: flex;
    flex-direction: row;
    align-items: center;
}

.anotacoes_label label:first-child {
    flex-grow: 1;
}

.anotacoes {
    display: flex;
    flex-direction: column;
    align-items: center;
}
</style>