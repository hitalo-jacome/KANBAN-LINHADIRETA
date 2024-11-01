<template>
    <div class="kanban-board container-fluid">
        <div class="board-header text-center">
            <h1>Gestão de Demandas</h1>
            <button @click="openNewCardModal" class="btn btn-primary">Nova Demanda</button>
        </div>

        <div class="board-columns row justify-content-between mt-4">
            <div v-for="column in columns" :key="column.id" :class="['column', getColumnClass(column.status)]"
                class="col-lg-3 col-md-4 col-sm-6 col-12 mb-4">
                <div class="column-header d-flex justify-content-between align-items-center mb-2">
                    <h2>{{ column.name }}</h2>
                    <span class="badge bg-secondary">{{ column.cards.length }}</span>
                </div>

                <draggable v-model="column.cards" group="cards" :animation="200" ghost-class="ghost-card"
                    :itemKey="card => card.id" @change="onDragChange">
                    <template #item="{ element: card }">
                        <div :class="['card', getCardClass(card.statusId)]" class="mb-2 p-2"
                            @click="openCardDetails(card)">
                            <div class="card-header d-flex justify-content-between">
                                <span>{{ card.titulo }}</span>
                                <span :class="getStatusClass(card.statusId)">{{ getStatusClass(card.statusId)
                                    }}</span>
                            </div>
                            <div class="card-body">
                                <p class="card-description">{{ card.descricao }}</p>
                                <div class="card-metadata d-flex justify-content-between">
                                    <span v-if="card.responsavelId">
                                        <i class="fas fa-user"></i> {{ getUserById(card.responsavelId) }}
                                    </span>
                                    <span class="protocol">#{{ card.protocolo }}</span>
                                </div>
                            </div>
                            <div class="card-footer d-flex justify-content-between">
                                <span class="date-info">
                                    <i class="fas fa-calendar"></i> Recebido: {{ formatDate(card.dataRecebimento) }}
                                </span>
                                <span class="date-info" v-if="card.prazo">
                                    <i class="fas fa-clock"></i> Prazo: {{ formatDate(card.prazo) }}
                                </span>
                            </div>
                        </div>
                    </template>
                </draggable>

            </div>
        </div>

        <!-- Modal para Nova Demanda -->
        <div v-if="showNewCardModal" class="custom-modal-overlay">
            <div class="custom-modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Nova Demanda</h5>
                    <button type="button" class="btn-close" @click="closeNewCardModal"></button>
                </div>
                <hr>
                <div class="modal-body">
                    <form @submit.prevent="createCard">
                        <div class="mb-3">
                            <label class="form-label">Protocolo</label>
                            <input v-model="newCard.Protocolo" class="form-control" placeholder="Número do protocolo"
                                required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Título</label>
                            <input v-model="newCard.Titulo" class="form-control" placeholder="Título da demanda"
                                required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Descrição</label>
                            <textarea v-model="newCard.Descricao" class="form-control"
                                placeholder="Descrição detalhada da demanda" rows="3"></textarea>
                        </div>

                        <div class="row">
                            <div class="mb-3 col-md-6">
                                <label class="form-label">Responsável</label>
                                <select v-model="newCard.ResponsavelId" class="form-select">
                                    <option value="">Selecione...</option>
                                    <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}
                                    </option>
                                </select>
                            </div>

                            <div class="mb-3 col-md-6">
                                <label class="form-label">Prazo</label>
                                <input type="date" v-model="newCard.Prazo" class="form-control">
                            </div>
                        </div>

                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" @click="closeNewCardModal">Cancelar</button>
                            <button type="submit" class="btn btn-primary">Salvar</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable';
import axios from 'axios';

export default {
    name: 'KanbanBoard',
    components: {
        draggable
    },
    data() {
        return {
            columns: [
                { id: 1, name: 'Recebida', status: 1, cards: [] },
                { id: 2, name: 'Análise Presidência', status: 2, cards: [] },
                { id: 3, name: 'Tratativa Responsável', status: 3, cards: [] },
                { id: 4, name: 'Análise Final', status: 4, cards: [] },
                { id: 5, name: 'Retorno ao Cliente', status: 5, cards: [] }
            ],
            showNewCardModal: false,
            newCard: this.getEmptyCard(),
            users: [
                { id: 1, name: 'João Silva' },
                { id: 2, name: 'Maria Santos' },
                { id: 3, name: 'Pedro Oliveira' }
            ]
        };
    },
    mounted() {
        this.loadCards();
    },
    methods: {
        // Modal control methods
        openNewCardModal() {
            this.showNewCardModal = true;
        },
        closeNewCardModal() {
            this.showNewCardModal = false;
        },

        // Helper methods
        getStatusClass(statusId) {
            const statusMap = {
                1: 'RECEBIDA',
                2: 'ANALISE_PRESIDENCIA',
                3: 'TRATATIVA_RESPONSAVEL',
                4: 'ANALISE_FINAL',
                5: 'COMPLETA'
            };
            return statusMap[statusId] || 'UNKNOWN';
        },
        getUserById(responsavelId) {
            const user = this.users.find(user => user.id === responsavelId);
            return user ? user.name : 'Desconhecido';
        },
        getEmptyCard() {
            return {
                Protocolo: '',
                Titulo: '',
                Descricao: '',
                ResponsavelId: '',
                Prazo: null,
                DataRecebimento: new Date().toISOString().split('T')[0],
                StatusId: 1
            };
        },
        formatDate(date) {
            if (!date) return '';
            return new Date(date).toLocaleDateString('pt-BR');
        },

        // Styling methods
        getColumnClass(status) {
            const classMap = {
                1: 'column-recebida',
                2: 'column-analise',
                3: 'column-tratativa',
                4: 'column-final',
                5: 'column-completa'
            };
            return classMap[status] || 'column-default';
        },
        getCardClass(statusId) {
            const classMap = {
                1: 'card-recebida',
                2: 'card-analise',
                3: 'card-tratativa',
                4: 'card-final',
                5: 'card-completa'
            };
            return classMap[statusId] || 'card-default';
        },

        // API call methods
        async loadCards() {
            try {
                const response = await axios.get('https://localhost:7025/api/LinhaDireta');
                // Map the cards to their respective columns
                this.columns = this.columns.map(column => ({
                    ...column,
                    cards: response.data.filter(card => card.statusId === column.status)
                }));
            } catch (error) {
                console.error('Erro ao carregar demandas:', error);
            }
        },
        async createCard() {
            try {
                console.log(this.newCard);
                await axios.post('https://localhost:7025/api/LinhaDireta', this.newCard);
                this.loadCards();
            } catch (error) {
                console.error('Erro ao criar demanda:', error.response.data);
            }
        },
        async onDragChange(event) {
            const { added } = event;
            if (added) {
                const card = added.element;
                const newStatus = this.columns.find(col => col.cards.includes(card)).status;
                card.statusId = newStatus;
                try {
                    await axios.put(`https://localhost:7025/api/LinhaDireta/${card.id}/${newStatus}`);
                } catch (error) {
                    console.error('Erro ao mover demanda:', error);
                    await this.loadCards(); // Recarrega os cards em caso de erro
                }
            }
        },

        // Placeholder for card details method
        openCardDetails() { }
    }
};
</script>


<style scoped>
/* Estilo geral do quadro Kanban */
.kanban-board {
    height: 100vh;
    background: #f5f6fa;
    padding-top: 1em;
}

/* Estilo das colunas do Kanban */
.board-columns {
    display: flex;
    gap: 0;
    overflow-x: auto;
    padding: 20px 0;
}

.column {
    background: #ffffff;
    border-radius: 10px;
    min-width: 360px;
    max-width: 360px;
    height: fit-content;
    padding: 16px;
    margin: 0 2em;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.column-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
    padding-bottom: 8px;
    border-bottom: 2px solid #f0f0f0;
}

.card-count {
    background: #e9ecef;
    padding: 2px 8px;
    border-radius: 12px;
    font-size: 0.9em;
}

/* Estilo dos cartões dentro das colunas */
.card {
    background: white;
    border: 1px solid #e9ecef;
    border-radius: 8px;
    padding: 12px;
    margin-bottom: 12px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
    cursor: pointer;
}

.card:hover {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transform: translateY(-2px);
}

.card-info {
    margin: 8px 0;
}

.card-metadata {
    display: flex;
    justify-content: space-between;
    margin-top: 8px;
    font-size: 0.9em;
    color: #666;
}

/* Estilo específico para as cores das colunas */
.column-recebida {
    background-color: #e3f2fd;
    /* Azul claro */
    border: 1px solid #1976d2;
    /* Azul escuro */
}

.column-analise {
    background-color: #fff3e0;
    /* Laranja claro */
    border: 1px solid #f57c00;
    /* Laranja escuro */
}

.column-tratativa {
    background-color: #e8f5e9;
    /* Verde claro */
    border: 1px solid #388e3c;
    /* Verde escuro */
}

.column-final {
    background-color: #f3e5f5;
    /* Roxo claro */
    border: 1px solid #7b1fa2;
    /* Roxo escuro */
}

.column-completa {
    background-color: #e8eaf6;
    /* Azul acinzentado */
    border: 1px solid #3f51b5;
    /* Azul escuro */
}

/* Estilo dos formulários e campos */
.form-group {
    margin-bottom: 16px;
}

.form-row {
    display: flex;
    gap: 16px;
}

.form-row .form-group {
    flex: 1;
}

label {
    display: block;
    margin-bottom: 4px;
    font-weight: 500;
}

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

.custom-modal-content h5 {
    margin: 0;
}

.modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
}

hr {
    width: 100%;
    border: 0.5px solid #ddd;
}
</style>
