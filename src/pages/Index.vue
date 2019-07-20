<template>
  <q-page>
    <div class="row q-px-sm bg-blue-grey-5">
      <!-- ヘッダー (フォルダパス) -->
      <div class="col-xs-12">
        <q-input dense dark borderless :value="currentDir">
          <template v-slot:prepend>
            <q-icon name="folder" />
          </template>
          <template v-slot:append>
            <q-icon name="more_horiz" class="cursor-pointer"
              />
          </template>
        </q-input>
      </div>
    </div>
    <div class="row">
      <div class="col col-grow list-col">
        <!-- フィルターテキストボックス -->
        <!-- フィルターテキストボックス -->
        <div class="bg-blue-grey-6">
          <q-input dense dark borderless class="q-mx-sm"
            v-model="filterPattern"
            >
          </q-input>
        </div>
        <!-- ファイル一覧 -->
        <q-list dense>
          <q-item clickable
            v-for="item in displayItems" :key="`item_${item.name}`"
            @click="setCurrentItem(item)"
            >
            <q-item-section avatar>
              <q-icon
                :color="item.isFile ? 'blue-grey-3' : 'yellow-14'"
                :name="item.isFile ? 'insert_drive_file' : 'folder'"
                />
            </q-item-section>
            <q-item-section>{{ item.name }}</q-item-section>
          </q-item>
        </q-list>
      </div>
      <div class="col editor-col">
        <!-- ファイルパス -->
        <!-- エディター -->
        <div id="editor"></div>
      </div>
    </div>
  </q-page>
</template>

<style lang="stylus" scoped>
.q-page
  height 100%
  min-height 100%
  display flex
  flex-direction column
  > :nth-child(1)
    flex 0 0 auto
  > :last-child
    flex 1 0 0
.editor-col, .list-col
  display flex
  flex-direction column
  > :last-child
    flex 1 0 0
    overflow auto
.list-col
  width 30%
  max-width 30%
  border-right solid 1px $blue-grey-3
</style>

<script>
import fs from 'fs';
import path from 'path';
import 'ace-min-noconflict';

export default {
  name: 'PageIndex',
  data() {
    return {
      currentDir: process.cwd(), // 選択中のフォルダパス
      currentFile: null, // 選択中のファイル
      filterPattern: '', // 絞り込み用フィルター文字列
      list: [], // フォルダのアイテムの配列
      editor: null, // Ace エディターのオブジェクト
    };
  },
  computed: {
    displayItems() {
      if (!this.filterPattern) return this.list;
      const regex = new RegExp(this.filterPattern);
      return this.list.filter(x => regex.test(x.name));
    },
  },
  methods: {
    loadItems() {
      fs.readdir(this.currentDir, (err, files) => {
        if (err) throw err;
        this.list = files.map((file) => {
          const absolutePath = path.resolve(this.currentDir, file);
          const isFile = fs.statSync(absolutePath).isFile();
          return {
            name: file,
            absolutePath,
            isFile,
          };
        });
      });
    },
    setCurrentItem(item) {
      if (item.isFile) {
        this.currentFile = item;
        fs.readFile(item.absolutePath, 'utf8', (err, data) => {
          this.editor.setValue(data, -1);
        });
      }
    },
  },
  mounted() {
    this.editor = window.ace.edit('editor');
    this.loadItems();
  },
};
</script>
