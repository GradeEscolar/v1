<template>
    <section class="container" :class="{ edit: edit }" v-if="anotacao">
        <div class="subcontainer" :class="{ edit: edit }">
            <div v-if="exibirTitulos" class="titulo">
                <span>
                    <p>{{ dataCompleta }}</p>
                </span>

                <span class="buttons">
                    <span>
                        <p>
                            <span>{{ anotacao.aula }}ª aula - </span>
                            <span class="disciplina">{{ disciplina?.disciplina }}</span>
                        </p>
                    </span>

                    <span class="lnk" @click="definirEdit(true)" v-if="!edit">
                        <p class="no-print">Editar</p>
                    </span>

                    <span v-if="!edit && origem == 'aula'" class="no-print">|</span>
                    <span class="lnk" @click="anotacoes" v-if="!edit && origem == 'aula'">
                        <p class="no-print">
                            <i class="pi pi-book"></i>
                            Anotações
                        </p>
                    </span>

                    <span v-if="!edit && origem == 'anotacao'" class="no-print">|</span>
                    <span class="lnk" @click="aula" v-if="!edit && origem == 'anotacao'">
                        <p class="no-print">
                            <i class="pi pi-file-edit"></i>
                            Aula
                        </p>
                    </span>

                    <span class="lnk" @click="alternarPreview()" v-if="edit">
                        <p class="no-print">{{ preview ? 'Editar' : 'Visualizar' }}</p>
                    </span>
                    <span v-if="edit">|</span>
                    <span class="lnk" v-if="edit" @click="salvar()">
                        <p class="no-print">Salvar</p>
                    </span>
                    <span v-if="edit">|</span>
                    <span class="lnk" @click="definirEdit(false)" v-if="edit">
                        <p class="no-print">Cancelar</p>
                    </span>
                </span>

            </div>

            <div v-if="edit && !preview" class="conteudo_edit">
                <textarea id="conteudo" v-model.trim="conteudo_edit" autofocus ref="ta"></textarea>
                <span class="lnk" @click="help = !help">Como formatar esta anotação?</span>
                <span class="mk" v-if="help">
                    <br />
                    <br />
                    Adicione os caracteres especiais #, *, >, ~ e - ao seu texto da forma como é apresentado abaixo:
                    <br />
                    <br />
                    <h3>### Eventos (provas, entrega de trabalhos, etc...)</h3>
                    <h1># Título</h1>
                    <h2>## Subtítulo</h2>
                    <p>Texto em **<strong>Negrito</strong>**, *<em>Itálico</em>*, ~~<s>Tachado</s>~~.</p>
                    <blockquote>
                        <p>&gt; Citação</p>
                    </blockquote>
                    <ul>
                        <li>- Lista</li>
                        <li>- Lista</li>
                    </ul>
                    <p>--- Linha divisória</p>
                    <hr />
                    <br />

                    <h4>Tabelas</h4>
                    <p><b>Estrutura:</b></p>
                    <p style="margin-left: 10px; margin-bottom: 5px;">
                        Titulo das colunas<br />
                        Alinhamento das colunas (esquerda, centralizado e direita)<br />
                        Conteúdo da tabela
                    </p>
                    <p><b>Exemplo:</b></p>
                    <p style="margin-left: 10px; margin-bottom: 5px;">
                        Coluna 1 | Coluna 2 | Coluna 3<br />
                        :--- | :---: | ---:<br />
                        Valor 1 | Valor 2 | Valor 3
                    </p>
                    <p><b>Resultado:</b></p>
                    <table style="margin-left: 10px; margin-bottom: 5px;">
                        <caption></caption>
                        <header>
                            <tr>
                                <th style="text-align: left;">Coluna 1</th>
                                <th style="text-align: center;">Coluna 2</th>
                                <th style="text-align: right;">Coluna 3</th>
                            </tr>
                        </header>
                        <tbody>
                            <tr>
                                <td style="text-align: left;">Valor 1</td>
                                <td style="text-align: center;">Valor 2</td>
                                <td style="text-align: right;">Valor 3</td>
                            </tr>
                        </tbody>
                    </table>

                    <br />

                    <br />
                    <br />
                    <br />
                    <br />
                </span>
            </div>
            <div v-if="!edit || preview" class="conteudo_view" :class="{ borda: exibirTitulos, preview: preview }">
                <span v-html="conteudo_view" class="mk"
                    :class="{ sem_anotacao: conteudo_view == '-- sem anotação --' }"></span>
            </div>
        </div>
    </section>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Dia from '@/Models/Dia';
import Anotacao from '@/Models/Anotacao';
import MarkdownIt from 'markdown-it';
import AnotacaoService from '@/Services/AnotacaoService';
import Disciplina from '@/Models/Disciplina';


export default defineComponent({
    name: 'AnotacaoComponent',

    props: {
        anotacao: Anotacao,
        disciplina: Disciplina,
        exibirTitulos: Boolean,
        origem: String
    },

    data(): {
        service: AnotacaoService,
        edit: boolean,
        preview: boolean,
        dataCompleta: string | undefined,
        conteudo_view: string | undefined,
        conteudo_edit: string | undefined,
        help: boolean,
        md: MarkdownIt
    } {
        return {
            service: new AnotacaoService(this.axios),
            edit: false,
            preview: false,
            dataCompleta: undefined,
            conteudo_view: undefined,
            conteudo_edit: undefined,
            help: false,
            md: new MarkdownIt({
                xhtmlOut: true,
                breaks: true,
                linkify: true
            })
        }
    },

    emits: ['editando', 'goToPage'],

    methods: {
        definirDataCompleta(anotacao: Anotacao | undefined) {
            if (anotacao?.data) {
                let data = new Date(anotacao.data).toISOString().substring(0, 10);
                let dt = new Date(`${data}T00:00:00.000-03:00`);
                this.dataCompleta = Dia.dataCompleta(dt);
            } else {
                return undefined;
            }
        },
        definirConteudo(anotacao: Anotacao | undefined) {
            if (anotacao?.anotacao) {
                this.conteudo_edit = anotacao.anotacao;
                this.conteudo_view = this.md.render(this.conteudo_edit);
            } else {
                this.conteudo_edit = undefined;
                this.conteudo_view = '-- sem anotação --';
            }
        },
        definirEdit(edit: boolean) {
            this.definirConteudo(this.anotacao);
            this.edit = edit;
            this.$emit('editando', this.edit);
            this.preview = false;
            this.help = false;
        },
        alternarPreview() {
            this.preview = !this.preview;
            if (this.preview) {
                const conteudo = this.conteudo_edit == undefined || this.conteudo_edit == '' ? '-- sem anotação --' : this.md.render(this.conteudo_edit);
                this.conteudo_view = conteudo;
            }
        },
        async salvar() {
            this.anotacao!.anotacao = this.conteudo_edit;
            let anotacao_db = await this.service.salvarAnotacao(this.anotacao!);
            this.anotacao!.id = anotacao_db.id;
            this.definirEdit(false);
        },
        anotacoes() {
            if (this.disciplina)
                sessionStorage.setItem('anotacoes_disciplina', this.disciplina.id!.toString());

            if (this.anotacao?.data) {
                const data = new Date(this.anotacao.data).toISOString().substring(0, 10);
                const dt = new Date(`${data}T00:00:00.000-03:00`);
                const mes = Dia.obterMes(dt);
                sessionStorage.setItem('anotacoes_mes', mes);
            }
            
            this.$emit('goToPage', 'Anotacoes');
        },
        aula() {
            if(this.anotacao){
                const data = new Date(this.anotacao.data).toISOString().substring(0, 10);
                sessionStorage.setItem('aula_data', data);
                sessionStorage.setItem('aula_aula', this.anotacao.aula.toString());
            }

            this.$emit('goToPage', 'Aula');
        }
    },

    async mounted() {
        this.definirDataCompleta(this.anotacao);
        this.definirConteudo(this.anotacao);
    },

    watch: {
        anotacao(newAnotacao: Anotacao | undefined) {
            this.definirDataCompleta(newAnotacao);
            this.definirConteudo(newAnotacao);
        }
    }

})
</script>

<style scoped>
section.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 10px 5px 18px 5px;
}

section.container.edit {
    position: fixed;
    top: var(--header-height);
    width: 100vw;
    height: calc(100vh - var(--header-footer-height) - 10px);
    background-color: white;
    padding: 10px 0 0 0;
    overflow-y: auto;
}

div.subcontainer {
    width: 100%;
    max-width: 600px;
}

div.subcontainer.edit {
    width: calc(100% - 10px);
    padding: 0 5px 0px 5px;
}

.titulo {
    display: flex;
    flex-direction: column;
    font-size: 10pt;
}

.disciplina {
    font-weight: bold;
}

span p {
    margin: 0;
}

span.buttons {
    display: flex;
    flex-direction: row;
    align-items: end;
}

span.buttons span:first-child {
    flex-grow: 1;
}

.editar:hover {
    text-shadow: 0 0 1px var(--lnk-hover-color);
    text-decoration: underline;
    text-decoration-thickness: .01px;
}

.conteudo_view {
    margin: 5px 0 5px 0;
    padding: 6px;
}

.conteudo_edit {
    margin: 5px 0 0 0;
    padding: 0 3px 0 0;
    font-size: 10pt;
}

@media only screen and (hover: none) and (pointer: coarse) {
    .conteudo_view.preview {
        margin: 5px 0 65px 0;
        padding: 6px;
    }

    .conteudo_edit {
        margin: 5px 0 0 0;
        padding: 0 3px 0 0;
    }

}

.conteudo_view.borda {
    border: 1px solid var(--input-border-color);
    border-radius: 5px;
}

.sem_anotacao {
    font-size: 10pt;
    color: gray;
    font-style: italic;
}

textarea {
    font-family: var(--font-family);
    font-size: 14px;
    width: calc(100% - 10px);
    height: 350px;
    border: 1px solid var(--input-border-color);
    border-radius: 5px;
    padding: 6px;
}

span.help {
    padding: 5px 0 60px 10px;
}

@media print {
    .no-print {
        display: none;
    }
}
</style>