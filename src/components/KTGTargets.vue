<template>
    <div class="ktg-targets">

        <div class="container ktg-targets__container">
            
            <div class="ktg-targets__header my-4 pb-4">
                <div class="row align-items-center">
                    <div class="col-8 col-sm-4 col-md-3 col-lg-3 col-xl-2">
                        <div class="ktg-targets__header-logo">KAN-Targets</div>
                    </div>
                    <div class="col-4 col-sm-3 col-md-5 col-lg-6 col-xl-7">
                        <KTGLang />
                    </div>
                    <div class="col-12 col-sm-5 col-md-4 col-lg-3 col-xl-3">
                        <KTGCreateTarget />
                    </div>
                </div>
            </div>

            <div class="ktg-targets__search">
                <div class="inputWrapper">
                    <input 
                        class="form-control ktg-targets__searchInput inputWrapper__field" 
                        type="text" 
                        :placeholder="getTranslate.TARGETS_SEARCH + '...'" 
                        id="searchTargets" 
                        v-model="inputSearch"
                    >
                    <i class="fas fa-times-circle inputWrapper__icon" v-if="showInputSearchClean" @click.prevent="inputSearch = ''"></i>
                </div>
                <div class="alert alert-danger mt-4 ktg-targets__searchError" v-if="filteredTargetsEmpty">{{ filteredTargetsEmpty }}</div>
            </div>

            <div 
                class="alert alert-danger alert-dismissible mt-4 ktg-targets__message" 
                v-if="'success' in deleteTargetMsg && deleteTargetMsg.success"
            >
                {{ deleteTargetMsg.text }}
                <button type="button" class="btn-close" aria-label="Close" @click="setDeleteTargetMsg({})"></button>
            </div>

            <div class="ktg-targets__list my-4" v-if="Object.keys(targets.value).length">
                <div class="accordion ktg-targets__accordion" id="targetsList">
                    <template v-for="target in targets.value" :key="target.id">
                        <KTGTarget 
                            :target="target"
                            @delete-target-id="setDeleteTargetID($event)" 
                        />
                    </template>
                </div>
            </div>

            <div class="ktg-targets__empty my-4" v-if="!targets.value.length">
                {{ getTranslate.TARGETS_EMPTY }}
            </div>

            <KTGImportTargets />

        </div>

        <div class="modal fade" id="deleteTargetModal" tabindex="-1" aria-labelledby="deleteTargetModalLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="deleteTargetModalLabel">{{ getTranslate.TARGET_DELETE_HEADER }}</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        {{ getTranslate.TARGET_DELETE_ALERT }}.
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-light" data-bs-dismiss="modal">{{ getTranslate.BTN_CANCEL }}</button>
                        <button type="button" class="btn btn-danger" data-bs-dismiss="modal" @click="targetsStore.deleteTarget(deleteTargetID)">{{ getTranslate.BTN_DELETE }}</button>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useTargetsStore }      from '@/stores/targets'
import { useTranslaterStore }   from '@/stores/translater'

import KTGLang          from '@/components/KTGLang.vue'
import KTGTarget        from '@/components/KTGTarget.vue'
import KTGCreateTarget  from '@/components/KTGCreateTarget.vue'
import KTGImportTargets from '@/components/KTGImportTargets.vue'

const targetsStore      = useTargetsStore()
const translaterStore   = useTranslaterStore()

const getTranslate          = computed(() => translaterStore.getTranslate)
const storeTargets          = computed(() => targetsStore.targets)
const targets               = computed(() => filteredTargets.value.length ? filteredTargets : storeTargets)
const deleteTargetMsg       = computed(() => targetsStore.deleteTargetMsg)
const setDeleteTargetMsg    = data => targetsStore.setDeleteTargetMsg(data)

let deleteTargetID          = ref(),
    showInputSearchClean    = ref(false),
    filteredTargets         = ref([]),
    filteredTargetsEmpty    = ref(),
    inputSearch             = ref('')

const filterTargets = input => {

    filteredTargets.value = []
    filteredTargetsEmpty.value = null

    if(input && input.length > 2) {

        showInputSearchClean.value = true

        if(storeTargets.value.length) {

            for(let target of storeTargets.value) {

                const reg = new RegExp(input.toUpperCase(), 'g')

                if( reg.test(target.name.toUpperCase()) || reg.test(target.descr.toUpperCase()) ) {

                    filteredTargets.value.push(target)
                }
            }

            if(filteredTargets.value.length == 0) {

                filteredTargets.value = []
                filteredTargetsEmpty.value = getTranslate.value.ERROR_SEARCH_EMPTY
            }
        }

    } else {

        showInputSearchClean.value = false
        filteredTargets.value = []
        filteredTargetsEmpty.value = null
    }
}

const setDeleteTargetID = id => deleteTargetID.value = id

targetsStore.readTargets()

watch(inputSearch, async (text) => {

    filterTargets(text)
})
</script>

<style lang="scss">
@import '@/assets/styles/scss/components/targets.scss';
</style>