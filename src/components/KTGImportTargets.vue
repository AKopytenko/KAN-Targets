<template>
    <div class="ktg-card ktg-card_inline ktg-import">

        <div class="ktg-card__body ktg-import__body">

            <h3 class="ktg-import__header">{{ getTranslate.IMPORT_TITLE }}</h3>
            <div class="ktg-import__descr">{{ getTranslate.IMPORT_DESCR }}.</div>

            <div class="mt-3 btns ktg-import__buttons">

                <button 
                    type="button"
                    class="btn btn-primary mr-2 ktg-import__download"
                    :disabled="!storeTargets.length"
                    @click.prevent="downloadTargets()"
                >
                    {{ getTranslate.BTN_DOWNLOAD }}
                </button>

                <button 
                    type="button"
                    class="btn btn-primary mr-2 ktg-import__upload"
                    data-bs-toggle="modal" 
                    data-bs-target="#importTargetsModal" 
                    aria-hidden="true"
                    @click.prevent="importTargetsMsg = {}"
                >
                    {{ getTranslate.BTN_UPLOAD }}
                </button>

            </div>

        </div>

        <div class="modal fade ktg-import__modal" id="importTargetsModal" tabindex="-1" aria-labelledby="importTargetsModalLabel" aria-hidden="true">
            <div class="modal-dialog ktg-import__modalDialog">
                <div class="modal-content ktg-import__modalContent">
                    <div class="modal-header ktg-import__modalHeader">
                        <h5 class="modal-title ktg-import__modalTitle" id="importTargetsModalLabel">{{ getTranslate.IMPORT_HEADER }}</h5>
                        <button type="button" class="btn-close ktg-import__modalClose" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <form class="ktg-import__modalForm" action="/" method="post" id="importTargetsForm" @submit.prevent="uploadTargets($event)">
                        <div class="modal-body ktg-import__modalBody">
                            <div class="ktg-import__result" v-if="'success' in importTargetsMsg">
                                <div 
                                    class="alert"
                                    :class="{
                                        'alert-success' : importTargetsMsg.success,
                                        'alert-danger' : !importTargetsMsg.success
                                    }"
                                >
                                    {{ importTargetsMsg.text }}
                                </div>
                            </div>
                            <div class="ktg-import__error" v-if="formErrors.length">
                                <div class="alert alert-danger">
                                    <div v-for="error in formErrors" :key="error">
                                        {{ error }}
                                    </div>
                                </div>
                            </div>
                            <div class="mb-3 ktg-import__formGroup">
                                <label for="importTargetsMethod" class="form-label ktg-import__formLabel">{{ getTranslate.IMPORT_METHOD }}:</label>
                                <select 
                                    id="importTargetsMethod" 
                                    class="form-select ktg-import__formInput"
                                    :class="{ 'is-invalid': 'importTargetsMethod' in invalidFields }" 
                                    @change="$event.target.classList.remove('is-invalid')"
                                >
                                    <option value="" selected disabled>...</option>
                                    <option value="push">{{ getTranslate.IMPORT_ADD }}</option>
                                    <option value="rewrite">{{ getTranslate.IMPORT_REWRITE }}</option>
                                </select>
                                <div class="mt-3 mb-4 ktg-import__formDescr">
                                    <div class="my-2 ktg-import__formDescrItem">
                                        <strong>{{ getTranslate.IMPORT_ADD }}</strong> 
                                        {{ getTranslate.IMPORT_ADD_DESCR }}
                                    </div>
                                    <div class="my-2 ktg-import__formDescrItem">
                                        <strong>{{ getTranslate.IMPORT_REWRITE }}</strong> 
                                        {{ getTranslate.IMPORT_REWRITE_DESCR }}
                                    </div>
                                </div>
                            </div>
                            <div class="mb-3 ktg-import__formGroup">
                                <label for="importTargetsFile" class="form-label ktg-import__formLabel">{{ getTranslate.IMPORT_FILE_TITLE }}:</label>
                                <input
                                    type="file" 
                                    id="importTargetsFile" 
                                    class="form-control ktg-import__formInput" 
                                    accept=".txt"
                                    :class="{ 'is-invalid': 'importTargetsFile' in invalidFields }" 
                                    @change="$event.target.classList.remove('is-invalid')"
                                >
                            </div>
                        </div>
                        <div class="modal-footer ktg-import__modalFooter">
                            <button type="button" class="btn btn-light ktg-import__modalBtn" data-bs-dismiss="modal">{{ getTranslate.BTN_CANCEL }}</button>
                            <button type="submit" class="btn btn-primary ktg-import__modalBtn">{{ getTranslate.BTN_UPLOAD }}</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { useStore } from 'vuex'

export default {

    name: 'KTGImportTargets',

    setup() {

        const store = useStore()

        let invalidFields     = ref({}),
            formErrors        = ref([]),
            importTargetsMsg  = ref({})

        const storeTargets  = computed(() => store.state.targets.targets)
        const getTranslate  = computed(() => store.getters.getTranslate)
        const setTargets    = msg => store.commit('setTargets', msg)

        function downloadTargets(targets) {

            const targetsString = JSON.stringify(targets)
            const targetsFile = new Blob( [ targetsString ], {type: 'application/json'})
            
            let now     = new Date(),
                year    = now.getFullYear(),
                month   = ('0' + (Number(now.getMonth()) + 1)).slice(-2),
                day     = ('0' + now.getDate()).slice(-2),
                hours   = ('0' + now.getHours()).slice(-2),
                minutes = ('0' + now.getMinutes()).slice(-2),
                seconds = ('0' + now.getSeconds()).slice(-2)

            let a = document.createElement('a')

            a.href = URL.createObjectURL(targetsFile)
            a.download = 'KANTargetsImport__' + year + '-' + month + '-' + day + '_' + hours + '-' + minutes + '-' + seconds + '.txt'
            a.click()
        }

        function uploadTargets(event) {

            const self = this
            const fields = event.target.elements

            let data = {}

            invalidFields.value = {}
            formErrors.value = []

            if(fields.importTargetsMethod.value) {
                data.method = fields.importTargetsMethod.value
            } else {
                invalidFields.value.importTargetsMethod = true
                formErrors.value.push(this.getTranslate.ERROR_IMPORT_METHOD)
            }

            if(fields.importTargetsFile.value) {

                let file     = fields.importTargetsFile.files[0],
                    fileType = file.type,
                    fileExt  = file.name.split('.')[1]

                if( fileType == 'text/plain' || fileExt == 'txt' ) {
                    data.file = fields.importTargetsFile.files[0]
                } else {
                    invalidFields.value.importTargetsFile = true
                    formErrors.value.push(this.getTranslate.ERROR_IMPORT_FILE_FORMAT)
                }
            } else {
                console.log(1)
                invalidFields.value.importTargetsFile = true
                console.log(2)
                formErrors.value.push(this.getTranslate.ERROR_IMPORT_FILE)
            }

            if(formErrors.value.length == 0) {

                const reader = new FileReader()

                let newTargets = []

                switch( data.method ) {

                    case 'rewrite':

                        reader.readAsText(data.file)

                        reader.onload = function(event) {

                            try {

                                const fileContent = JSON.parse(event.target.result)

                                for(let target of fileContent) {

                                    if( 
                                        'id'        in target &&
                                        'name'      in target && 
                                        'descr'     in target && 
                                        'priority'  in target && 
                                        'created'   in target 
                                    ) {

                                        newTargets.push({
                                            
                                            id:         target.id,
                                            name:       target.name,
                                            descr:      target.descr,
                                            priority:   target.priority,
                                            created:    target.created
                                        })

                                    } else {

                                        importTargetsMsg = { success: false, text: self.getTranslate.ERROR_IMPORT_FILE_FORMAT }
                                        return false
                                    }
                                }

                            } catch {

                                importTargetsMsg = { success: false, text: self.getTranslate.ERROR_IMPORT_FILE_FORMAT }
                                return false
                            }

                            self.setTargets(newTargets)

                            importTargetsMsg = { success: true, text: self.getTranslate.IMPORT_SUCCESS }
                        }

                        break

                    case 'push':

                        reader.readAsText(data.file)

                        reader.onload = function(event) {

                            try {

                                const fileContent = JSON.parse(event.target.result)

                                let targetID = 0

                                if(storeTargets.value.length) {

                                    const allIDs = storeTargets.value.map( target => target.id )

                                    targetID = Math.max.apply(null, allIDs) + 1
                                }

                                for(let target of fileContent) {

                                    if(
                                        'id'        in target && 
                                        'name'      in target && 
                                        'descr'     in target && 
                                        'priority'  in target && 
                                        'created'   in target
                                    ) {
                                        
                                        console.log('push - Формат: ОК')

                                        newTargets.push({

                                            id: targetID, 
                                            name: target.name,
                                            descr: target.descr,
                                            priority: target.priority,
                                            created: target.created
                                        })

                                        targetID++

                                    } else {

                                        console.log('push - Формат: FAIL', target)

                                        importTargetsMsg = { success: false, text: self.getTranslate.ERROR_IMPORT_FILE_FORMAT }
                                        return false
                                    }
                                }

                            } catch {

                                importTargetsMsg = { success: false, text: self.getTranslate.ERROR_IMPORT_FILE_FORMAT }
                                return false
                            }

                            newTargets = [...storeTargets.value, ...newTargets]

                            self.setTargets(newTargets)

                            importTargetsMsg = { success: true, text: self.getTranslate.IMPORT_SUCCESS }
                        }

                        break
                }

            }
        }

        onMounted(() => {

            const importTargetsModal = document.getElementById('importTargetsModal')

            importTargetsModal.addEventListener('show.bs.modal', () => {

                formErrors.value = []
                invalidFields.value = {}
                importTargetsMsg.value = {}
                document.querySelector('#importTargetsForm').reset()
            })
        })

        return {

            invalidFields,
            formErrors,
            importTargetsMsg,
            storeTargets,
            getTranslate,
            setTargets,
            downloadTargets,
            uploadTargets
        }
    }
}
</script>

<style lang="scss">
@import '@/assets/styles/scss/components/import-targets';
</style>