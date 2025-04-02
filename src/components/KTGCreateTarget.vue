<template>
    <div class="ktg-createTarget">

        <a href="#" class="btn btn-primary mt-4 mt-sm-0 ktg-createTarget__btnModal" data-bs-toggle="modal" data-bs-target="#createTargetModal" aria-hidden="true">{{ getTranslate.BTN_NEW_TASK }}</a>

        <div class="modal fade ktg-createTarget__modalAdd" tabindex="-1" role="dialog" id="createTargetModal" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ getTranslate.TARGET_NEW_HEADER }}</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <form class="ktg-createTarget__form" action="/" method="post" @submit.prevent="sendTargetForm()" id="createTargetForm">
                        <div class="modal-body">
                            <div 
                                v-if="'success' in createTargetMsg"
                                class="mb-3 alert ktg-createTarget__message"
                                :class="{
                                    'alert-success' : createTargetMsg.success,
                                    'alert-danger' : !createTargetMsg.success
                                }"
                            >
                                {{ createTargetMsg.text }}
                            </div>
                            <div class="mb-3 ktg-createTarget__message" v-if="formErrors.length">
                                <div class="alert alert-danger">
                                    <div v-for="error in formErrors" :key="error">
                                        {{ error }}
                                    </div>
                                </div>
                            </div>
                            <div class="mb-3 ktg-createTarget__form-group">
                                <label for="targetName">{{ getTranslate.TARGET_NEW_NAME }}</label>
                                <input 
                                    type="text" 
                                    id="targetName" 
                                    class="form-control" 
                                    :class="{ 'is-invalid': 'targetName' in invalidFields }" 
                                    v-model="targetName"
                                    @change="checkValid($event)"
                                >
                            </div>
                            <div class="mb-3 ktg-createTarget__formform-group">
                                <label for="targetDescr">{{ getTranslate.TARGET_NEW_TEXT }}</label>
                                <textarea 
                                    id="targetDescr" 
                                    class="form-control" 
                                    :class="{ 'is-invalid': 'targetDescr' in invalidFields }" 
                                    v-model="targetDescr"
                                    rows="5" 
                                    @change="checkValid($event)"
                                ></textarea>
                            </div>
                            <div class="form-check ktg-createTarget__formform-group">
                                <input 
                                    type="checkbox" 
                                    id="targetPriority"
                                    class="form-check-input" 
                                    v-model="targetPriority"
                                >
                                <label class="form-check-label" for="targetPriority">{{ getTranslate.TARGET_NEW_PRIORITY }}</label>
                            </div>
                        </div>
                        <div class="modal-footer ktg-createTarget__btns">
                            <button type="button" class="btn btn-light" data-bs-dismiss="modal">{{ getTranslate.BTN_CANCEL }}</button>
                            <button type="submit" class="btn btn-primary">{{ getTranslate.BTN_ADD }}</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useTargetsStore }      from '@/stores/targets'
import { useTranslaterStore }   from '@/stores/translater'

const targetsStore      = useTargetsStore()
const translaterStore   = useTranslaterStore()

const getTranslate      = computed(() => translaterStore.getTranslate)
const createTargetMsg   = computed(() => targetsStore.createTargetMsg)

const createTarget          = data => targetsStore.createTarget(data)
const setCreateTargetMsg    = data => targetsStore.setCreateTargetMsg(data)

let targetName      = ref(''),
    targetDescr     = ref(''),
    targetPriority  = ref(false),
    invalidFields   = ref({}),
    formErrors      = ref([]),
    modalShow       = ref(false)

const sendTargetForm = () => {

    invalidFields.value = {}
    formErrors.value = []
    setCreateTargetMsg({})

    let data = {}

    if(targetName.value) {

        data.name = targetName.value

    } else {

        invalidFields.value.targetName = true
        formErrors.value.push(getTranslate.value.ERROR_TARGET_TITLE)
    }

    if(targetDescr.value) {

        data.descr = targetDescr.value

    } else {

        invalidFields.value.targetDescr = true
        formErrors.value.push(getTranslate.value.ERROR_TARGET_TEXT)
    }

    data.priority = targetPriority.value || false
    data.created = (Date.now() / 1000).toFixed()

    if(formErrors.value.length == 0) {

        createTarget(data)
    }
}

const checkValid = event => {

    event.target.classList.remove('is-invalid')
}

watch(createTargetMsg, async msg => {

    if('success' in msg && msg.success) {

        targetName.value        = '',
        targetDescr.value       = '',
        targetPriority.value    = false,
        modalShow.value         = false
    }
})

watch(modalShow, async () => {

    if(!modalShow.value) {

        targetName.value      = ''
        targetDescr.value     = ''
        targetPriority.value  = false

        setCreateTargetMsg({})
    }
})

</script>

<style lang="scss">
@import '@/assets/styles/scss/components/create-target.scss';
</style>