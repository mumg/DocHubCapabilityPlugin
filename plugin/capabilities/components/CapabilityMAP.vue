<template>
  <div class="capability-map">
    <capability-root-group
      v-for="c0 in capabilityMap"
      v-bind:id="c0.id"
      v-bind:key="c0.id"
      v-bind:list="c0.list"
      v-bind:title="c0.title" />
  </div>
</template>

<script>
  import CapabilityRootGroup from './CapabilityRootGroup.vue';
  import get_size from 'get-size';
  import Masonry from 'masonry-layout';
  import potpack from 'potpack';
  export default {
    name: 'CapabilitiesMAP',
    components: {
      CapabilityRootGroup
    },
    props: {
      profile: {
        type: Object,
        required: true
      },
      getContent: {
        type: Function,
        required: true
      },
      pullData: {
        type: Function,
        required: true
      },
      path: {
        type: String,
        required: true
      },
      params: {
        type: Object,
        default: null
      },
      toPrint: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        refresher: null,
        capabilityMap: [],
        elementsTree: []
      };
    },

    watch: {
      profile() {
        this.onRefresh();
      }
    },
    mounted() {
      this.onRefresh();
    },
    methods: {
      doLayout(){
        for ( const g of this.elementsTree ){
          const elements = [];
          g.children.forEach((c)=>{
            const e = document.getElementById(c);
            let sz = get_size(e);
            if (  sz.height % 50 > 0 ){
              let height = (Math.floor(sz.height / 50) + 1) * 50;
              e.style.height = height.toString() + 'px';
              sz.height = height;
            }
            elements.push(({
              w: sz.width + 10,
              h: sz.height + 10,
              data: e
            }));
          });
          if ( g.depth === 1 ){
            const container = document.getElementById('container-' + g.id);
            g.masonry = new Masonry(container, { columnWidth: 50, gutter: 10});
          }else if ( g.depth > 1){
            const group = document.getElementById(g.id);
            potpack(elements);
            const container = document.getElementById('container-' + g.id);
            const title = group.getElementsByClassName('capability-group-title');
            let width = 0;
            let height = 0;
            elements.forEach((c)=>{
              if ( c.x + c.w  > width ){
                width = c.x + c.w;
              }
              if ( c.y + c.h > height){
                height = c.y + c.h;
              }
            });
            group.style.width = (width + 20).toString() + 'px';
            const tsz = get_size(title[0]);
            group.style.height = ( 10 + tsz.height + height).toString() + 'px';
            container.style.width = (width + 10).toString() + 'px';
            container.style.height = (5 + tsz.height + height).toString() + 'px';
            elements.forEach((c)=>{
              c.data.style.left = (c.x + 5).toString() + 'px';
              c.data.style.top = (c.y + 5 + tsz.height).toString() + 'px';
            });
          }
        }
      },
      doRefresh() {
        if (this.profile) {
          this.pullData(this.profile.source).then((result) => {
            class cg {
              constructor() {
                this.m = new Map();
              }

              has(key) {
                return this.m.has(key);
              }

              get(key) {
                return this.m.get(key);
              }

              set(key, val) {
                this.m.set(key, val);
              }

              size() {
                return this.m.size;
              }

              keys() {
                return this.m.keys();
              }

              values() {
                return this.m.values();
              }

              get_first() {
                return this.m.entries().next().value[1];
              }
            }

            let map = new cg();

            function append(path, val, func, create) {
              const p = path.split('.').reverse();
              let m = map;
              while (p.length > 0) {
                const l = p.pop();
                if (m.has(l)) {
                  m = m.get(l);
                } else {
                  if (create) {
                    const _m = new cg();
                    m.set(l, _m);
                    m = _m;
                  } else {
                    func(m, val);
                    return;
                  }
                }
              }
              func(m, val);
            }

            for (const grp of result.filter((v) => v.type === 'group')) {
              append(grp.id, grp, (m, val) => {
                m.title = val.title;
                m.id = val.id;
              }, true);
            }

            for (const grp of result.filter((v) => v.type !== 'group')) {
              append(grp.id, grp, (m, val) => {
                if (!m.children) {
                  m.children = [];
                }
                m.children.push(val);
              }, false);
            }

            function compact(m) {
              while (m.size() === 1 && !m.title) {
                m = m.get_first();
              }
              for (let _m of m.keys()) {
                let f = m.get(_m);
                m.set(_m, compact(f));
              }
              return m;
            }

            map = compact(map);

            const levels = [
              'none',
              'low',
              'moderate',
              'high'
            ];

            function capability(c) {
              let items = [];
              let level = levels.length - 1;
              if (c.list && c.list.length > 0) {
                for (const s of c.list) {
                  let idx = levels.indexOf(s.level);
                  if (idx >= 0 && level > idx) {
                    level = idx;
                  }
                  items.push({
                    'title': s.title,
                    'id': s.id
                  });
                }
              } else {
                level = 0;
              }
              return {
                'type': 'capability',
                'id': c.id,
                'title': c.title,
                'list': items,
                'level': levels[level]
              };
            }

            function convert(m) {
              let items = [];
              for (const _m of m.values()) {
                items.push(convert(_m));
              }
              if (m.children) {
                for (const c of m.children) {
                  items.push(capability(c));
                }
              }

              return {
                'type': 'group',
                'id': m.id,
                'title': m.title,
                'list': items
              };
            }

            function process(m) {
              let items = [];
              for (const _m of m.values()) {
                items.push(convert(_m));
              }
              return items;
            }

            function tree(m, t, idx) {
              if (idx === undefined) {
                idx = 0;
              } else {
                idx = idx + 1;
              }
              let _t = t;
              if (_t === undefined) {
                _t = [];
              }
              const node = {
                id: m.id,
                children: [],
                depth: idx
              };
              for (const _m of m.values()) {
                node.children.push(_m.id);
                tree(_m, _t, idx);
              }
              if (m.children) {
                for (const c of m.children) {
                  node.children.push(c.id);
                }
              }
              if ( idx !== 0 ){
                _t.push(node);
              }
              return _t;
            }

            this.capabilityMap = process(map);
            this.elementsTree = tree(map);
            this.$nextTick(() => {
              this.doLayout();
            });
          });
        }
      },

      onRefresh() {
        if (this.refresher) clearTimeout(this.refresher);
        this.refresher = setTimeout(this.doRefresh, 50);
      }
    }
  };
</script>

<style scoped>
.capability-map {
  display: grid;
  align-content: center
}

</style>
