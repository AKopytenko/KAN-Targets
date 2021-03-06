<template>
    <div class="ktg-targets">
        
        <div class="container ktg-targets__container">
            
            <div class="ktg-targets__header my-4 pb-4">
                <div class="row align-items-center">
                    <div class="col-12 col-sm-6">
                        <div class="ktg-targets__headerLogo">KAN-Targets</div>
                    </div>
                    <div class="col-12 col-sm-6">
                        <CreateTarget />
                    </div>
                </div>
            </div>

            <div class="ktg-targets__search">

                <div class="inputWrapper">
                    <input 
                        class="form-control ktg-targets__searchInput inputWrapper__field" 
                        type="text" 
                        placeholder="Поиск задач..." 
                        id="searchTargets" 
                        v-model="inputSearch"
                    >
                    <i class="fas fa-times-circle inputWrapper__icon" v-if="searchTargetsIcon" @click.prevent="clearSearchInput"></i>
                </div>
                
                <div class="alert alert-danger mt-4 ktg-targets__searchError" v-if="filteredTargetsEmpty">{{ filteredTargetsEmpty }}</div>
            </div>

            <div class="ktg-targets__list my-4" v-if="targets.length">
                <div class="accordion ktg-targets__accordion" id="targetsList">
                    <template v-for="target in targets">
                        <Target 
                            :target="target"
                            :key="target.id"
                            @delete-target-id="setDeleteTargetID($event)" 
                        />
                    </template>
                </div>
            </div>

            <div class="ktg-targets__empty my-4" v-if="!targets.length">
                Список задач пуст...
            </div>

            <ImportTargets v-if="targets" />

        </div>

        <div class="modal fade" id="deleteTargetModal" tabindex="-1" aria-labelledby="deleteTargetModalLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="deleteTargetModalLabel">Удаление задачи</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        Выбранная задача будет удалена без возможности восстановления.
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-light" data-bs-dismiss="modal">Отмена</button>
                        <button type="button" class="btn btn-danger" data-bs-dismiss="modal" @click="deleteTarget(deleteTargetID)">УДАЛИТЬ</button>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script>

import 'bootstrap'

import { mapGetters, mapActions } from 'vuex'

import Target           from './Target'
import CreateTarget     from './CreateTarget'
import ImportTargets    from './ImportTargets'

export default {
    name: 'Targets',
    data() {

        return {
            
            deleteTargetID: null,

            filteredTargets: [],
            filteredTargetsEmpty: null,
            
            inputSearch: '',
            searchTargetsIcon: false
        }
    },
    components: {

        Target,
        CreateTarget,
        ImportTargets
    },
    computed: {

        ...mapGetters([

            'getTargets'
        ]),

        targets() {

            if(this.filteredTargets.length) {
                return this.filteredTargets
            } else {
                return this.getTargets
            }
        }
    },
    methods: {

        ...mapActions([

            'readTargets', 
            'deleteTarget'
        ]),

        clearSearchInput() {

            document.querySelector('#searchTargets').value = ''
            this.searchTargetsIcon = false
            this.filterTargets()
        },

        filterTargets(input) {

            this.filteredTargets = []
            this.filteredTargetsEmpty = null

            if(input && input.length > 2) {

                this.searchTargetsIcon = true

                if(this.getTargets.length) {

                    for(let target of this.getTargets) {

                        const reg = new RegExp(input.toUpperCase(), 'g')

                        if( reg.test(target.name.toUpperCase()) || reg.test(target.descr.toUpperCase()) ) {
                            
                            this.filteredTargets.push(target)
                        }
                    }

                    if(this.filteredTargets.length == 0) {

                        this.filteredTargets = []
                        this.filteredTargetsEmpty = 'Нет задач, содержащих указанные данные.'
                    }
                }
            } else {

                this.filteredTargets = []
                this.filteredTargetsEmpty = null
                this.searchTargetsIcon = false
            }
        },

        setDeleteTargetID(id) {

            this.deleteTargetID = id
        }
    },
    mounted() {

        this.readTargets()
    },
    watch: {

        inputSearch(text) {

            this.filterTargets(text)
        }
    }
}
</script>

<style lang="scss">
@import '~bootstrap/dist/css/bootstrap.min.css';
@import '~@fortawesome/fontawesome-free/css/all.min.css';
@import '@/assets/styles/layout.scss';
</style>
