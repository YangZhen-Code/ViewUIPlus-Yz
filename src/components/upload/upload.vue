<template>
    <div :class="[prefixCls]">
        <div :class="[prefixCls + '-wrap']">
            <ul
                class="image-list"
                v-if="showType === 'Image' && imageList.length && multiple"
            >
                <template v-for="item in imageList" :key="item">
                    <li
                        :style="{
                            width: imageWidth + 'px',
                            height: imageHeight + 'px',
                        }"
                    >
                        <Image
                            :src="item"
                            preview
                            :preview-list="imageList"
                        ></Image>
                    </li>
                </template>
            </ul>
            <div
                :class="classes"
                @click="handleClick"
                @drop.prevent="onDrop"
                @paste="handlePaste"
                @dragover.prevent="dragOver = true"
                @dragleave.prevent="dragOver = false"
            >
                <input
                    ref="input"
                    type="file"
                    :class="[prefixCls + '-input']"
                    @change="handleChange"
                    :multiple="multiple"
                    :webkitdirectory="webkitdirectory"
                    :accept="accept"
                />

                <div
                    :class="[prefixCls + '-drag', prefixCls + '-imageType']"
                    v-if="showType === 'Image'"
                >
                    <Icon
                        v-if="fileList.length === 0 || !multiple"
                        type="ios-add"
                        size="62"
                    ></Icon>
                    <Image
                        v-if="
                            !multiple &&
                            fileList.length > 0 &&
                            fileList[0].status === 'finished'
                        "
                        :src="fileList[0].response.result.filePath"
                        preview
                        :preview-list="imageList"
                    ></Image>
                </div>
                <slot></slot>
            </div>
        </div>

        <slot name="tip"></slot>
        <upload-list
            v-if="showUploadList"
            :files="fileList"
            @on-file-remove="handleRemove"
            @on-file-preview="handlePreview"
        ></upload-list>
    </div>
</template>
<script>
import UploadList from "./upload-list.vue";
import ajax from "./ajax";
import { oneOf } from "../../utils/assist";
import mixinsForm from "../../mixins/form";

const prefixCls = "ivu-upload";

export default {
    name: "Upload",
    mixins: [mixinsForm],
    components: { UploadList },
    props: {
        action: {
            type: String,
            required: true,
        },
        headers: {
            type: Object,
            default() {
                return {};
            },
        },
        multiple: {
            type: Boolean,
            default: false,
        },
        data: {
            type: Object,
        },
        name: {
            type: String,
            default: "file",
        },
        withCredentials: {
            type: Boolean,
            default: false,
        },
        showUploadList: {
            type: Boolean,
            default: true,
        },
        type: {
            type: String,
            validator(value) {
                return oneOf(value, ["select", "drag"]);
            },
            default: "select",
        },
        format: {
            type: Array,
            default() {
                return [];
            },
        },
        accept: {
            type: String,
        },
        maxSize: {
            type: Number,
        },
        beforeUpload: Function,
        onProgress: {
            type: Function,
            default() {
                return {};
            },
        },
        onSuccess: {
            type: Function,
            default() {
                return {};
            },
        },
        onError: {
            type: Function,
            default() {
                return {};
            },
        },
        onRemove: {
            type: Function,
            default() {
                return {};
            },
        },
        onPreview: {
            type: Function,
            default() {
                return {};
            },
        },
        onExceededSize: {
            type: Function,
            default() {
                return {};
            },
        },
        onFormatError: {
            type: Function,
            default() {
                return {};
            },
        },
        defaultFileList: {
            type: Array,
            default() {
                return [];
            },
        },
        paste: {
            type: Boolean,
            default: false,
        },
        disabled: {
            type: Boolean,
            default: false,
        },
        webkitdirectory: {
            type: Boolean,
            default: false,
        },
        // 上传组件类型
        showType: {
            type: String,
            validator(value) {
                return oneOf(value, ["Image", "Video", "File", "Custom"]);
            },
            default: "File",
        },
        // imageWidth
        imageWidth: {
            type: Number,
            default: 100,
        },
        imageHeight: {
            type: Number,
            default: 100,
        },
    },
    data() {
        return {
            prefixCls: prefixCls,
            dragOver: false,
            fileList: [],
            tempIndex: 1,
            imageDisabled: false,
            imageList: [],
        };
    },
    computed: {
        classes() {
            return [
                `${prefixCls}`,
                {
                    [`${prefixCls}-select`]: this.type === "select",
                    [`${prefixCls}-drag`]: this.type === "drag",
                    [`${prefixCls}-dragOver`]:
                        this.type === "drag" && this.dragOver,
                },
            ];
        },
    },
    methods: {
        handleClick() {
            if (this.itemDisabled || this.imageDisabled) return;
            this.$refs.input.click();
        },
        handleChange(e) {
            const files = e.target.files;
            console.log("files", files);
            if (!files) {
                return;
            }
            this.uploadFiles(files);
            this.$refs.input.value = null;
        },
        onDrop(e) {
            this.dragOver = false;
            if (this.itemDisabled) return;
            this.uploadFiles(e.dataTransfer.files);
        },
        handlePaste(e) {
            if (this.itemDisabled) return;
            if (this.paste) {
                this.uploadFiles(e.clipboardData.files);
            }
        },
        uploadFiles(files) {
            let postFiles = Array.prototype.slice.call(files);
            if (!this.multiple) postFiles = postFiles.slice(0, 1);

            if (postFiles.length === 0) return;

            postFiles.forEach((file) => {
                this.upload(file);
            });
        },
        upload(file) {
            if (!this.beforeUpload) {
                return this.post(file);
            }

            const before = this.beforeUpload(file);
            if (before && before.then) {
                before.then(
                    (processedFile) => {
                        if (
                            Object.prototype.toString.call(processedFile) ===
                            "[object File]"
                        ) {
                            this.post(processedFile);
                        } else {
                            this.post(file);
                        }
                    },
                    () => {
                        // this.$emit('cancel', file);
                    }
                );
            } else if (before !== false) {
                this.post(file);
            } else {
                // this.$emit('cancel', file);
            }
        },
        post(file) {
            // check format
            if (this.format.length) {
                const _file_format = file.name
                    .split(".")
                    .pop()
                    .toLocaleLowerCase();
                const checked = this.format.some(
                    (item) => item.toLocaleLowerCase() === _file_format
                );
                if (!checked) {
                    this.onFormatError(file, this.fileList);
                    return false;
                }
            }

            // check maxSize
            if (this.maxSize) {
                if (file.size > this.maxSize * 1024) {
                    this.onExceededSize(file, this.fileList);
                    return false;
                }
            }

            this.handleStart(file);
            let formData = new FormData();
            formData.append(this.name, file);

            ajax({
                headers: this.headers,
                withCredentials: this.withCredentials,
                file: file,
                data: this.data,
                filename: this.name,
                action: this.action,
                onProgress: (e) => {
                    this.handleProgress(e, file);
                },
                onSuccess: (res) => {
                    this.handleSuccess(res, file);
                },
                onError: (err, response) => {
                    this.handleError(err, response, file);
                },
            });
        },
        handleStart(file) {
            file.uid = Date.now() + this.tempIndex++;
            const _file = {
                status: "uploading",
                name: file.name,
                size: file.size,
                percentage: 0,
                uid: file.uid,
                showProgress: true,
            };

            this.fileList.push(_file);
        },
        getFile(file) {
            const fileList = this.fileList;
            let target;
            fileList.every((item) => {
                target = file.uid === item.uid ? item : null;
                return !target;
            });
            return target;
        },
        handleProgress(e, file) {
            const _file = this.getFile(file);
            this.onProgress(e, _file, this.fileList);
            _file.percentage = e.percent || 0;
        },
        handleSuccess(res, file) {
            const _file = this.getFile(file);
            if (this.showType === "Image") {
                this.imageDisabled = true;
                this.imageList.push(res.result.filePath);
            }

            if (_file) {
                _file.status = "finished";
                _file.response = res;

                this.onSuccess(res, _file, this.fileList);
                this.handleFormItemChange("change", _file);
                console.log(this.fileList);
                setTimeout(() => {
                    _file.showProgress = false;
                }, 1000);
            }
        },
        handleError(err, response, file) {
            const _file = this.getFile(file);
            const fileList = this.fileList;

            _file.status = "fail";

            fileList.splice(fileList.indexOf(_file), 1);

            this.onError(err, response, file);
        },
        handleRemove(file) {
            const fileList = this.fileList;
            fileList.splice(fileList.indexOf(file), 1);
            this.onRemove(file, fileList);
            this.imageDisabled = false;
        },
        handlePreview(file) {
            if (file.status === "finished") {
                this.onPreview(file);
            }
        },
        clearFiles() {
            this.fileList = [];
        },
    },
    watch: {
        defaultFileList: {
            immediate: true,
            handler(fileList) {
                this.fileList = fileList.map((item) => {
                    item.status = "finished";
                    item.percentage = 100;
                    item.uid = Date.now() + this.tempIndex++;
                    return item;
                });
            },
        },
    },
};
</script>
