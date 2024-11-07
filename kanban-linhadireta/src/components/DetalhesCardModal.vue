<template>
    <div v-if="show" class="custom-modal-overlay">
            <div class="custom-modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Detalhes da Demanda</h5>
                    <button type="button" class="btn-close" @click="close"></button>
                </div>
                <hr>
                <div class="modal-body">
                    <h6>Título: {{ localCard.titulo }}</h6>
                    <p><strong>Descrição:</strong> {{ localCard.descricao }}</p>
                    <p><strong>Protocolo:</strong> {{ localCard.protocolo }}</p>
                    <p><strong>Responsável:</strong> {{ getUserById(localCard.responsavelId) }}</p>
                    <p><strong>Status Atual:</strong> {{ getStatusClass(localCard.statusId) }}</p>
                    <p><strong>Data de Recebimento:</strong> {{ formatDate(localCard.dataRecebimento) }}</p>
                    <p><strong>Prazo:</strong> {{ formatDate(localCard.prazo) }}</p>

                    <h6 class="mt-4">Histórico:</h6>
                    <ul>
                        <li v-for="entry in localCard.historico" :key="entry.id">
                            <p>
                                <strong>Data:</strong> {{ formatDate(entry.data) }}<br>
                                <strong>Status:</strong> {{ getStatusClass(entry.statusId) }}<br>
                                <strong>Usuário:</strong> {{ entry.usuario }}<br>
                                <strong>Observação:</strong> {{ entry.observacao }}
                            </p>
                        </li>
                    </ul>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="close">Fechar</button>
                </div>
            </div>
        </div>
</template>

<script>
export default {
    props: {
        show: {
            type: Boolean,
            required: true
        },
        card: {
            type: Object,
            required: true
        },
        getUserById: {
            type: Function,
            required: true
        },
        getStatusClass: {
            type: Function,
            required: true
        },
        formatDate: {
            type: Function,
            required: true
        }
    },
    data() {
        return {
            localCard: { ...this.card } // Cria uma cópia local da prop `card`
        };
    },
    watch: {
        // Atualiza a cópia local sempre que a prop `card` mudar
        card: {
            handler(newValue) {
                this.localCard = { ...newValue };
            },
            deep: true,
            immediate: true
        }
    },
    methods: {
        close() {
            this.$emit('close');
        }
    }
};
</script>

<style scoped>
/* Estilo para o modal customizado */
.custom-modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}
.custom-modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 10px;
    width: 43em;
    max-width: 100%;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    position: relative;
}
.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 1.25rem;
    margin-bottom: 1rem;
}
.btn-close {
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
}
</style>