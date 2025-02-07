<template>
    <div>
        <!-- Alerta de erro -->
        <div v-if="errorMessage" class="uk-alert-danger" uk-alert>
            <a class="uk-alert-close" uk-close></a>
            <p>{{errorMessage}}</p>
        </div>      

        <!-- Slider com atividades da semana -->
        <div uk-slider class="uk-background-muted uk-margin-bottom">
            <div class="uk-position-relative uk-padding uk-padding-remove-bottom uk-visible-toggle uk-light" tabindex="-1">
                <!-- Lista de Atividades -->
                <h4 class="uk-text-danger">Atividades da Semana</h4>
                <ul class="uk-slider-items uk-child-width-1-1 uk-child-width-1-2@s uk-child-width-1-3@l uk-grid">
                    <template v-if="atividadesDaSemana.length > 0">
                        <li v-for="(atv, index) in atividadesDaSemana" :key="index">
                            <!-- Card de Atividade -->
                            <div class="uk-card uk-card-secondary">
                                <div class="uk-card-body">
                                    <h3 class="uk-card-title uk-text-truncate uk-margin-remove">{{atv.titulo}}</h3>
                                    <small class="uk-text-left">Data: {{atv.data_entrega | formatDate}}</small>
                                    <p class="uk-text-truncate">{{atv.descricao}}</p>
                                </div>
                                <div class="uk-card-footer">
                                    <router-link :to="{ name: atv.route, params: {id: atv.id}}" class="uk-position-bottom-right uk-button uk-button-danger">
                                        Veja mais...
                                    </router-link>
                                </div>
                            </div>
                            <!--  -->
                        </li>
                    </template>
                    <li v-else style="color: black !important">
                        Nenhuma atividade para essa semana ainda...
                    </li>
                </ul>
                <!--  -->
                <!-- Botões de previous e next -->
                <a class="uk-position-center-left uk-background-muted uk-text-danger uk-box-shadow-large uk-border-rounded uk-position-small uk-hidden-hover" href="#" uk-slidenav-previous
                    uk-slider-item="previous"></a>
                <a class="uk-position-center-right uk-background-muted uk-text-danger uk-box-shadow-large uk-border-rounded uk-position-small uk-hidden-hover" href="#" uk-slidenav-next
                    uk-slider-item="next"></a>
                <!--  -->
            </div>
            <!-- Navegador de pontos do slide -->
            <ul class="uk-slider-nav uk-dotnav uk-flex-center uk-margin"></ul>
            <!--  -->
        </div>
        <!--  -->

        <!-- Calendário de Eventos -->
        <ul uk-accordion>
            <li class="uk-open  uk-background-muted">
                <a class="uk-accordion-title uk-button-accordion-li uk-background-secondary uk-text-white uk-padding-small" href="#">Calendário</a>
                <div class="uk-accordion-content">
                    <FullCalendar 
                        ref="fullCalendar"
                        :weekNumbers="true"
                        :weekNumbersWithinDays="true"
                        :customButtons="buttonsCalendar"
                        :selectable="true"
                        :header="headerCalendar"
                        defaultView="dayGridMonth"
                        :plugins="calendarPlugins"
                        locale="pt-BR"
                        :buttonText="{today: 'hoje'}"
                        :events="atividades"
                    />
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction';

export default {
    data() {
        return {
            errorMessage: null,
            calendarPlugins: [ dayGridPlugin, interactionPlugin ],
            buttonsCalendar: {
                customPrevButton: {
                    icon: 'fc-icon-chevron-right',
                    click: ()=> {
                        let calendarApi = this.$refs.fullCalendar.getApi();
                        this.getAtividadesDoMes(calendarApi.getDate().getMonth() + 2, calendarApi.getDate().getFullYear(), calendarApi);                        
                    }
                },
                customNextButton: {
                    icon: 'fc-icon-chevron-left',
                    click: ()=> {
                        let calendarApi = this.$refs.fullCalendar.getApi();
                        this.getAtividadesDoMes(calendarApi.getDate().getMonth(), calendarApi.getDate().getFullYear(), calendarApi, true);                        
                    }
                },
            },
            headerCalendar: {
                left: 'title  ',
                center: '',
                right: 'customNextButton,customPrevButton'
            },
            atividades: [],
            atividadesDaSemana: []
        }
    },
    components: {
        FullCalendar
    },
    methods: {
        async getAtividadesDoMes(mes, ano, calendarApi, isPrevious = false){
            try {
                this.errorMessage = null;

                // Recupera o token
                let token = this.$session.get('jwt');

                // Salva os cursos e as matérias
                let atividades = await this.$http.get(process.env.API_URL + `atividade/by-month/${mes}/${ano}`, {headers: {Authorization: token}});
                this.atividades = [];

                atividades.body.data.atividadesCurso.forEach(atv => {
                    this.atividades.push({title: atv.titulo, date: atv.data_entrega, route: 'verAtividadeCurso'});
                });
                atividades.body.data.atividadesLazer.forEach(atv => {
                    this.atividades.push({title: atv.titulo, date: atv.data_entrega, route: 'verAtividadeLazer'});
                });
                atividades.body.data.atividadesMateria.forEach(atv => {
                    this.atividades.push({title: atv.titulo, date: atv.data_entrega, route: 'verAtividadeMateria'});
                });

                if(isPrevious)
                    calendarApi.prev();
                else
                    calendarApi.next();
            }
            catch(e){
                throw e;
                // this.errorMessage = e.body.message;
            }   
        },
        async getAtividadesDaSemana(){
            try {
                let calendarApi = this.$refs.fullCalendar.getApi();
                let data = calendarApi.getNow();
                this.errorMessage = null;

                // Recupera o token
                let token = this.$session.get('jwt');

                // Salva os cursos e as matérias
                let atividades = await this.$http.get(process.env.API_URL + 'atividade/by-week/' + data, {headers: {Authorization: token}});
                this.atividadesDaSemana = [];

                atividades.body.data.atividadesCurso.forEach(atv => {
                    this.atividadesDaSemana.push({titulo: atv.titulo, data_entrega: atv.data_entrega, descricao: atv.descricao, id: atv.atividadesCurso[0].id, route: 'verAtividadeCurso'});
                });
                atividades.body.data.atividadesLazer.forEach(atv => {
                    this.atividadesDaSemana.push({titulo: atv.titulo, data_entrega: atv.data_entrega, descricao: atv.descricao, id: atv.atividadesLazer[0].id, route: 'verAtividadeLazer'});
                });
                atividades.body.data.atividadesMateria.forEach(atv => {
                    this.atividadesDaSemana.push({titulo: atv.titulo, data_entrega: atv.data_entrega, descricao: atv.descricao, id: atv.atividadesMateria[0].id, route: 'verAtividadeMateria'});
                });
            }
            catch(e){
                this.errorMessage = e.body.message;
            }  
        }
    },
    async mounted() {
        try {
            this.getAtividadesDaSemana();
            
            let calendarApi = this.$refs.fullCalendar.getApi();
            let mes = calendarApi.getDate().getMonth() + 1;
            let ano = calendarApi.getDate().getFullYear();
            this.errorMessage = null;

            // Recupera o token
            let token = this.$session.get('jwt');

            // Salva os cursos e as matérias
            let atividades = await this.$http.get(process.env.API_URL + `atividade/by-month/${mes}/${ano}`, {headers: {Authorization: token}});
            this.atividades = [];

            atividades.body.data.atividadesCurso.forEach(atv => {
                this.atividades.push({title: atv.titulo, date: atv.data_entrega});
            });
            atividades.body.data.atividadesLazer.forEach(atv => {
                this.atividades.push({title: atv.titulo, date: atv.data_entrega});
            });
            atividades.body.data.atividadesMateria.forEach(atv => {
                this.atividades.push({title: atv.titulo, date: atv.data_entrega});
            });

            calendarApi.render();
        }
        catch(e){
            this.errorMessage = e.body.message;
        }  
    }
}
</script>

<style scoped>
@import '~@fullcalendar/core/main.css';
@import '~@fullcalendar/daygrid/main.css';

.uk-card-body{
    padding: 20px;
}
</style>