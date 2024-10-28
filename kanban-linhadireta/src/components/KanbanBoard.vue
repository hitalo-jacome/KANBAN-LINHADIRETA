<template>
    <div class="kanban-board">
        <div class="board-header">
            <h1>Gestão de Demandas</h1>
            <button @click="openNewCardModal" class="btn-new">Nova Demanda</button>
        </div>

        <div class="board-columns">
            <div v-for="column in columns" :key="column.id" class="column">
                <div class="column-header">
                    <h2>{{ column.name }}</h2>
                    <span class="card-count">{{ column.cards.length }}</span>
                </div>

                <draggable v-model="column.cards" group="cards" :animation="200" ghost-class="ghost-card"
                    @change="onDragChange">
                    <template #item="{ element: card }">
                        <div class="card" @click="openCardDetails(card)">
                            <div class="card-header">
                                <span class="card-title">{{ card.title }}</span>
                                <span :class="'status-badge ' + getStatusClass(card.status)">
                                    {{ card.status }}
                                </span>
                            </div>
                            <div class="card-info">
                                <p class="card-description">{{ card.description }}</p>
                                <div class="card-metadata">
                                    <span class="responsible" v-if="card.responsible">
                                        <i class="fas fa-user"></i> {{ card.responsible }}
                                    </span>
                                    <span class="protocol">
                                        #{{ card.protocol }}
                                    </span>
                                </div>
                            </div>
                            <div class="card-footer">
                                <span class="date-info">
                                    <i class="fas fa-calendar"></i>
                                    Recebido: {{ formatDate(card.receivedDate) }}
                                </span>
                                <span class="date-info" v-if="card.deadline">
                                    <i class="fas fa-clock"></i>
                                    Prazo: {{ formatDate(card.deadline) }}
                                </span>
                            </div>
                        </div>
                    </template>
                </draggable>
            </div>
        </div>

        <!-- Modal para Nova Demanda -->
        <div v-if="showNewCardModal" class="modal">
            <div class="modal-content">
                <h2>Nova Demanda</h2>
                <form @submit.prevent="createCard">
                    <div class="form-group">
                        <label>Protocolo</label>
                        <input v-model="newCard.protocol" placeholder="Número do protocolo" required>
                    </div>

                    <div class="form-group">
                        <label>Título</label>
                        <input v-model="newCard.title" placeholder="Título da demanda" required>
                    </div>

                    <div class="form-group">
                        <label>Descrição</label>
                        <textarea v-model="newCard.description" placeholder="Descrição detalhada da demanda" rows="3">
                </textarea>
                    </div>

                    <div class="form-row">
                        <div class="form-group">
                            <label>Responsável</label>
                            <select v-model="newCard.responsible">
                                <option value="">Selecione...</option>
                                <option v-for="user in users" :key="user.id" :value="user.name">
                                    {{ user.name }}
                                </option>
                            </select>
                        </div>

                        <div class="form-group">
                            <label>Prazo</label>
                            <input type="date" v-model="newCard.deadline">
                        </div>
                    </div>

                    <div class="modal-actions">
                        <button type="button" @click="showNewCardModal = false">Cancelar</button>
                        <button type="submit">Salvar</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>

<script>
import draggable from 'vuedraggable'
import axios from 'axios'

export default {
    name: 'KanbanBoard',
    components: {
        draggable
    },
    data() {
        return {
            columns: [
                {
                    id: 1,
                    name: 'Recebida',
                    status: 'RECEIVED',
                    cards: []
                },
                {
                    id: 2,
                    name: 'Análise Presidência',
                    status: 'PRESIDENT_ANALYSIS',
                    cards: []
                },
                {
                    id: 3,
                    name: 'Tratativa Responsável',
                    status: 'IN_TREATMENT',
                    cards: []
                },
                {
                    id: 4,
                    name: 'Análise Final',
                    status: 'FINAL_ANALYSIS',
                    cards: []
                },
                {
                    id: 5,
                    name: 'Retorno ao Cliente',
                    status: 'COMPLETED',
                    cards: []
                }
            ],
            showNewCardModal: false,
            newCard: this.getEmptyCard(),
            users: [
                { id: 1, name: 'João Silva' },
                { id: 2, name: 'Maria Santos' },
                { id: 3, name: 'Pedro Oliveira' }
            ]
        }
    },
    methods: {
        getEmptyCard() {
            return {
                protocol: '',
                title: '',
                description: '',
                responsible: '',
                deadline: null,
                receivedDate: new Date().toISOString().split('T')[0],
                status: 'RECEIVED'
            }
        },

        async loadCards() {
            try {
                const response = await axios.get('/api/demands')
                // Distribuir os cards nas colunas corretas baseado no status
                this.columns = this.columns.map(column => ({
                    ...column,
                    cards: response.data.filter(card => card.status === column.status)
                }))
            } catch (error) {
                console.error('Erro ao carregar demandas:', error)
            }
        },

        async onDragChange({ added }) {
            if (added) {
                const card = added.element
                const newStatus = this.columns.find(col =>
                    col.cards.includes(card)
                ).status

                try {
                    await axios.put(`/api/demands/${card.id}/status`, {
                        status: newStatus
                    })
                    // Atualizar histórico de movimentação
                    await axios.post(`/api/demands/${card.id}/history`, {
                        status: newStatus,
                        date: new Date(),
                        user: 'Usuário Atual' // Implementar usuário logado
                    })
                } catch (error) {
                    console.error('Erro ao mover demanda:', error)
                    await this.loadCards()
                }
            }
        },

        getStatusClass(status) {
            const classes = {
                'RECEIVED': 'status-new',
                'PRESIDENT_ANALYSIS': 'status-analysis',
                'IN_TREATMENT': 'status-treatment',
                'FINAL_ANALYSIS': 'status-final',
                'COMPLETED': 'status-completed'
            }
            return classes[status] || 'status-default'
        },

        formatDate(date) {
            if (!date) return ''
            return new Date(date).toLocaleDateString('pt-BR')
        },

        // ... outros métodos existentes
    }
}
</script>

<style scoped>
.kanban-board {
    padding: 20px;
    height: 100vh;
    background: #f5f6fa;
}

.board-columns {
    display: flex;
    gap: 20px;
    overflow-x: auto;
    padding: 20px 0;
}

.column {
    background: #ffffff;
    border-radius: 10px;
    min-width: 320px;
    max-width: 320px;
    height: fit-content;
    padding: 16px;
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

.card {
    background: white;
    border: 1px solid #e9ecef;
    border-radius: 8px;
    padding: 12px;
    margin-bottom: 12px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
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

.status-badge {
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.8em;
    font-weight: 500;
}

.status-new {
    background: #e3f2fd;
    color: #1976d2;
}

.status-analysis {
    background: #fff3e0;
    color: #f57c00;
}

.status-treatment {
    background: #e8f5e9;
    color: #388e3c;
}

.status-final {
    background: #f3e5f5;
    color: #7b1fa2;
}

.status-completed {
    background: #e8eaf6;
    color: #3f51b5;
}

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

/* ... restante dos estilos existentes ... */
</style>
