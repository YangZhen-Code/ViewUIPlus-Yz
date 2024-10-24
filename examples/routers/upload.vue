<template>
    <Upload multiple type="drag" paste action="//120.79.147.193:3004/upload">
        <div style="padding: 20px 0">
            <Icon
                type="ios-cloud-upload"
                size="52"
                style="color: #3399ff"
            ></Icon>
            <p>Click or drag files here to upload</p>
        </div>
    </Upload>
    <Upload
        paste
        action="//120.79.147.193:3004/upload"
        @on-success="getFiles"
        showType="Image"
        multiple
    >
    </Upload>
</template>
<script>
export default {
    methods: {
        getFiles(res, file, fileList) {},
    },
};
</script>
