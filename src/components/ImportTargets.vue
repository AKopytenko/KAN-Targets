<template>
    <div class="ktg-card ktg-card_inline ktg-import">

        <div class="ktg-card__body ktg-import__body">

            <h3 class="ktg-import__header">Импорт/экспорт</h3>
            <div class="ktg-import__descr">Загрузить/cкачать задачи для переноса между устройствами.</div>

            <div class="mt-3 btns ktg-import__buttons">

                <button 
                    type="button"
                    class="btn btn-primary mr-2 ktg-import__download"
                    :disabled="!getTargets.length"
                    @click.prevent="downloadTargets()"
                >
                    Скачать
                </button>

                <button 
                    type="button"
                    class="btn btn-primary mr-2 ktg-import__upload"
                    data-bs-toggle="modal" 
                    data-bs-target="#importTargetsModal" 
                    aria-hidden="true"
                    @click.prevent="importTargetsMsg = null"
                >
                    Загрузить
                </button>

            </div>

        </div>

        <div class="modal fade ktg-import__modal" id="importTargetsModal" tabindex="-1" aria-labelledby="importTargetsModalLabel" aria-hidden="true">
            <div class="modal-dialog ktg-import__modalDialog">
                <div class="modal-content ktg-import__modalContent">
                    <div class="modal-header ktg-import__modalHeader">
                        <h5 class="modal-title ktg-import__modalTitle" id="importTargetsModalLabel">Загружить задачи из файла</h5>
                        <button type="button" class="btn-close ktg-import__modalClose" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <form class="ktg-import__modalForm" action="/" method="post" id="importTargetsForm" @submit.prevent="uploadTargets($event)">
                        <div class="modal-body ktg-import__modalBody">
                            <div class="ktg-import__result" v-if="importTargetsMsg">
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
                                <label for="importTargetsMethod" class="form-label ktg-import__formLabel">Способ загрузки:</label>
                                <select 
                                    id="importTargetsMethod" 
                                    class="form-select ktg-import__formInput"
                                    :class="{ 'is-invalid': 'importTargetsMethod' in invalidFields }" 
                                    @change="checkValid($event)"
                                >
                                    <option value="" selected disabled>...</option>
                                    <option value="push">Добавить</option>
                                    <option value="rewrite">Перезаписать</option>
                                </select>
                                <div class="mt-3 mb-4 ktg-import__formDescr">
                                    <div class="my-2 ktg-import__formDescrItem">
                                        <strong>Добавить</strong> 
                                        Задачи из файла будут добавлены к уже существующим.
                                    </div>
                                    <div class="my-2 ktg-import__formDescrItem">
                                        <strong>Перезаписать</strong> 
                                        Существующие задачи будут УДАЛЕНЫ, после загрузки останутся только задачи из файла.
                                    </div>
                                </div>
                            </div>
                            <div class="mb-3 ktg-import__formGroup">
                                <label for="importTargetsFile" class="form-label ktg-import__formLabel">Файл задач:</label>
                                <input
                                    type="file" 
                                    id="importTargetsFile" 
                                    class="form-control ktg-import__formInput" 
                                    accept=".txt"
                                    :class="{ 'is-invalid': 'importTargetsFile' in invalidFields }" 
                                    @change="checkValid($event)"
                                >
                            </div>
                        </div>
                        <div class="modal-footer ktg-import__modalFooter">
                            <button type="button" class="btn btn-light ktg-import__modalBtn" data-bs-dismiss="modal">Отмена</button>
                            <button type="submit" class="btn btn-primary ktg-import__modalBtn">Загрузить</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex'

export default {

    name: 'ImportTargets',

    data() {

        return {

            invalidFields: {},
            formErrors: [],
            importTargetsMsg: null
        }
    },

    computed: {

        ...mapGetters([

            'getTargets'
        ])
    },

    methods: {

        ...mapMutations([

            'setTargets'
        ]),

        downloadTargets() {

            const targetsString = JSON.stringify(this.getTargets)
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
        },

        async uploadTargets(event) {

            const self = this
            const fields = event.target.elements

            let data = {}

            this.invalidFields = {}
            this.formErrors = []

            if(fields.importTargetsMethod.value) {
                data.method = fields.importTargetsMethod.value
            } else {
                this.invalidFields.importTargetsMethod = true
                this.formErrors.push('Не выбран способ загрузки')
            }

            if(fields.importTargetsFile.value) {
                
                let file     = fields.importTargetsFile.files[0],
                    fileType = file.type,
                    fileExt  = file.name.split('.')[1]

                if( fileType == 'text/plain' || fileExt == 'txt' ) {
                    data.file = fields.importTargetsFile.files[0]
                } else {
                    this.invalidFields.importTargetsFile = true
                    this.formErrors.push('Неверный формат файла')
                }
            } else {
                this.invalidFields.importTargetsFile = true
                this.formErrors.push('Не выбран файл с задачами')
            }

            if(this.formErrors.length == 0) {

                const reader = new FileReader()

                let targets = []

                switch( data.method ) {

                    case 'rewrite':
                        
                        reader.readAsText(data.file)

                        reader.onload = function(event) {

                            try {

                                const fileContent = JSON.parse(event.target.result)

                                for(let target of fileContent) {

                                    if(
                                        'name'      in target && 
                                        'descr'     in target && 
                                        'priority'  in target && 
                                        'completed' in target &&
                                        'created'   in target && 
                                        'id'        in target
                                    ) {

                                        targets.push({

                                            name: target.name,
                                            descr: target.descr,
                                            priority: target.priority,
                                            completed: target.completed,
                                            created: target.created,
                                            id: target.id
                                        })
                                    }
                                }
                            } catch {

                                self.importTargetsMsg = { success: false, text: 'Неверный формат файла!' }
                                return false
                            }

                            self.setTargets(targets)

                            self.importTargetsMsg = { success: true, text: 'Задачи успешно загружены' }
                        }

                        break

                    case 'push':

                        reader.readAsText(data.file)

                        reader.onload = function(event) {

                            try {

                                const fileContent = JSON.parse(event.target.result)

                                let targetID = 0

                                if(self.getTargets.length) {

                                    const allIDs = self.getTargets.map( target => target.id )

                                    targetID = Math.max.apply(null, allIDs) + 1
                                }

                                for(let target of fileContent) {

                                    if(
                                        'name'      in target && 
                                        'descr'     in target && 
                                        'priority'  in target && 
                                        'completed' in target &&
                                        'created'   in target
                                    ) {

                                        targets.push({

                                            name: target.name,
                                            descr: target.descr,
                                            priority: target.priority,
                                            completed: target.completed,
                                            created: target.created,
                                            id: targetID
                                        })

                                        targetID++
                                    }
                                }
                            } catch {

                                self.importTargetsMsg = { success: false, text: 'Неверный формат файла!' }
                                return false
                            }

                            targets = [...self.getTargets, ...targets]

                            self.setTargets(targets)

                            self.importTargetsMsg = { success: true, text: 'Задачи успешно загружены' }
                        }

                        break
                }

            }
        },

        checkValid(event) {

            event.target.classList.remove('is-invalid')
        }
    },

    mounted() {

        this.formErrors = []
        this.invalidFields = {}
        this.importTargetsMsg = null
        document.querySelector('#importTargetsForm').reset()
    }
}
</script>

<style lang="scss">
@import '@/assets/styles/scss/components/ImportTargets.scss';
</style>