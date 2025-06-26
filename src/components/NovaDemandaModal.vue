<template>
    <div v-if="show" class="custom-modal-overlay">
        <div class="custom-modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Nova Demanda</h5>
                <button type="button" class="btn-close" @click="close"></button>
            </div>
            <hr>
            <div class="modal-body">
                <!-- Select Inicial para escolher Cliente ou Colaborador -->
                <div class="mb-3">
                    <label class="form-label">Tipo de Usuário</label>
                    <select v-model="tipoUsuario" class="form-select">
                        <option value="">Selecione...</option>
                        <option value="Cliente">Cliente</option>
                        <option value="Colaborador">Colaborador</option>
                    </select>
                </div>

                <!-- Formulário -->
                <div v-if="tipoUsuario">
                    <div v-if="tipoUsuario === 'Cliente'" class="row">
                        <input type="hidden" v-model="formData.tipoUsuario" value="Cliente">
                        <div class="mb-3 col-md-12">
                            <label class="form-label">Titulo <span class="text-danger">*</span></label>
                            <input v-model="formData.titulo" class="form-control" required>
                        </div>
                        <div class="mb-3 col-md-6">
                            <label class="form-label">Nome Completo <span class="text-danger">*</span></label>
                            <input v-model="formData.nomeCompleto" class="form-control" required>
                        </div>
                        <div class="mb-3 col-md-6">
                            <label class="form-label">Grupo <span class="text-danger">*</span></label>
                            <input v-model="formData.grupo" class="form-control" required>
                        </div>
                        <div class="mb-3 col-md-6">
                            <label class="form-label">Celular <span class="text-danger">*</span></label>
                            <input v-model="formData.celular" type="tel" class="form-control" required>
                        </div>
                        <div class="mb-3 col-md-6">
                            <label class="form-label">Email <span class="text-danger">*</span></label>
                            <input v-model="formData.email" type="email" class="form-control" required>
                        </div>
                        <!-- Documento -->
                        <div class="mb-3 col-md-4">
                            <label class="form-label">Tipo de Documento<span class="text-danger">*</span></label>
                            <div class="d-flex justify-content-center align-items-center">
                                <label class="form-check-label me-3">
                                    <input v-model="formData.tipoDocumento" type="radio" value="CPF"
                                        class="form-check-input" required>
                                    CPF
                                </label>
                                <label class="form-check-label">
                                    <input v-model="formData.tipoDocumento" type="radio" value="CNPJ"
                                        class="form-check-input" required>
                                    CNPJ
                                </label>
                            </div>
                        </div>
                        <div class="mb-3 col-md-6">
                            <label class="form-label">Número do Documento<span class="text-danger">*</span></label>
                            <input v-mask="'##.###.###/####-##'" v-model="formData.numeroDocumento"
                                class="form-control" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Mensagem<span class="text-danger">*</span></label>
                            <textarea v-model="formData.mensagem" class="form-control" rows="3" required></textarea>
                        </div>
                    </div>

                    <!-- Campos específicos de colaborador -->
                    <div v-if="tipoUsuario === 'Colaborador'">
                        <input type="hidden" v-model="formData.tipoUsuario" value="Colaborador">
                        <div class="mb-3">
                            <label class="form-label">Departamento</label>
                            <input v-model="formData.departamento" class="form-control">
                        </div>
                    </div>
                </div>

                <!-- Botões -->
                <div class="modal-footer d-flex justify-content-between">
                    <button type="button" class="btn btn-secondary" @click="close">Cancelar</button>
                    <button type="submit" class="btn btn-primary" :disabled="!formIsValid"
                        @click="createCard">Salvar</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { VueMaskDirective } from 'v-mask';
import axios from 'axios';

export default {
    directives: {
        mask: VueMaskDirective
    },
    props: {
        show: {
            type: Boolean,
            required: true
        }
    },
    data() {
        return {
            tipoUsuario: '',
            formData: {
                tipoUsuario: '',
                titulo: '',
                nomeCompleto: '',
                grupo: '',
                celular: '',
                email: '',
                tipoDocumento: '',
                numeroDocumento: '',
                mensagem: '',
                departamento: '',
            },
        };
    },
    computed: {
        formIsValid() {
            return this.formData.titulo && this.formData.mensagem && this.tipoUsuario;
        }
    },
    methods: {
        close() {
            this.$emit('close');
            this.resetForm();
        },
        async createCard() {
            try {
                await axios.post('https://localhost:7097/api/LinhaDireta', {
                    ...this.formData,
                    Protocolo: '',
                    StatusId: 1,
                    ResponsavelId: 1
                });
                this.$emit('refreshCards');
            } catch (error) {
                console.error('Erro ao criar demanda:', error.response?.data || error.message);
            }
        },
        resetForm() {
            this.tipoUsuario = '';
            this.formData = {
                tipoUsuario: '',
                titulo: '',
                nomeCompleto: '',
                grupo: '',
                celular: '',
                email: '',
                tipoDocumento: '',
                numeroDocumento: '',
                mensagem: '',
                departamento: '',
            };
        }
    }
};
</script>

<style scoped>
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
