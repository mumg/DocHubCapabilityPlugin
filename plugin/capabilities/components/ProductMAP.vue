<template>
  <div id="product-capability-map" class="product-capability-map">
    <div class="product-capability-header">
    </div>
    <div id="product-list" class="product-list">
      <product
        v-for="p in products"
        v-bind:id="p.id"
        v-bind:key="p.id"
        v-bind:title="p.title"
        v-bind:type="p.type"
        v-bind:capabilities="p.capabilities" />
    </div>
    <div id="product-capability-body" class="product-capability-body">
      <product-capability-group
        v-for="c0 in capabilityMap"
        v-bind:id="c0.id"
        v-bind:key="c0.id"
        v-bind:type="c0.type"
        v-bind:list="c0.list"
        v-bind:title="c0.title"
        v-bind:depth="c0.depth" />
    </div>
  </div>
</template>

<script>
  import get_size from 'get-size';
  import ProductCapabilityGroup from './ProductCapabilityGroup.vue';
  import Product from './Product.vue';
  export default {
    name: 'ProductMAP',
    components: {
      ProductCapabilityGroup,
      Product
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
        products: [],
        tree: [],
        margin: 0
      };
    },

    watch: {
      profile() {
        this.onRefresh();
      }
    },
    mounted() {
      this.onRefresh();
      window.addEventListener('resize', this.onResize);
    },
    beforeDestroy() {
      window.removeEventListener('resize', this.onResize);
    },
    methods: {
      onResize(){
        const l = document.getElementById('product-list');
        const map = document.getElementById('product-capability-body');
        const map_sz = get_size(map);
        l.style.width = (map_sz.width-this.margin - 15).toString() +'px';
      },
      doLayout(){
        const p = document.getElementById('product-capability-map').getBoundingClientRect();
        const l = document.getElementById('product-list');
        const _cache = new Map();
        function getElement(id){
          if ( _cache.has(id)){
            return _cache.get(id);
          }
          const el = document.getElementById(id);
          if (el){
            _cache.set(id, el);
          }
          return el;
        }
        function getOffset(el) {
          const rect = el.getBoundingClientRect();
          return {
            left: rect.left - p.left,
            top: rect.top - p.top
          };
        }
        const caps = new Set();
        this.products.forEach( (p)=>{
          p.capabilities.forEach((c)=>{
            caps.add(c.id);
          });
        });
        let left = 0;
        caps.forEach((c)=>{
          const el = getElement('container-' + c);
          if ( el){
            const off = getOffset(el);
            if ( off.left > left ){
              left = off.left;
            }
          }
        });
        left += 10;
        l.style.marginLeft = left.toString() + 'px';
        caps.forEach((c)=>{
          let height = 50;
          const el = getElement(c);
          this.products.forEach((p)=>{
            let found = false;
            p.capabilities.forEach((pc)=>{
              if (pc.id === c){
                found = true;
              }
            });
            if ( found){
              const el = getElement(p.id + '-' + c);
              const sz = get_size(el);
              if ( sz.height > height){
                height = sz.height;
              }
            }
          });

          el.style.height = (height+20).toString() + 'px';

        });
        const map = getElement('product-capability-body');
        const map_sz = get_size(map);
        for ( const p of this.products ){
          const product = getElement(p.id);
          product.style.height = (map_sz.height + 90).toString() + 'px';
        }
        this.$nextTick(()=>{
          caps.forEach((c)=>{
            const el = getElement(c);
            const of = getOffset(el);
            this.products.forEach((p)=>{
              let found = false;
              p.capabilities.forEach((pc)=>{
                if (pc.id === c){
                  found = true;
                }
              });
              if ( found){
                const el = getElement(p.id + '-' + c);
                el.style.top = (of.top-2).toString() + 'px';
              }
            });
          });
          l.style.height = (map_sz.height + 90).toString() + 'px';
          this.margin = left;;
          l.style.width = (map_sz.width-this.margin - 15).toString() +'px';
        });

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

            for (const grp of result.capabilities.filter((v) => v.type === 'group')) {
              append(grp.id, grp, (m, val) => {
                m.title = val.title;
                m.id = val.id;
              }, true);
            }

            for (const grp of result.capabilities.filter((v) => v.type !== 'group')) {
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

            function capability(c, idx) {
              let systems = [];
              let level = levels.length - 1;
              if (c.list && c.list.length > 0) {
                for (const s of c.list) {
                  let idx = levels.indexOf(s.level);
                  if (idx >= 0 && level > idx) {
                    level = idx;
                  }
                  systems.push({
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
                'systems': systems,
                'depth': idx,
                'list': [],
                'level': levels[level]
              };
            }

            function convert(m, idx) {
              if (idx === undefined ){
                idx = 0;
              }
              let items = [];
              for (const _m of m.values()) {
                let gr = convert(_m, idx + 1);
                if ( gr !== null ){
                  items.push(gr);
                }
              }
              if (m.children) {
                for (const c of m.children) {
                  if ( c.list?.length > 0 ){
                    items.push(capability(c, idx + 1));
                  }
                }
              }
              if ( items.length === 0){
                return null;
              }
              return {
                'type': 'group',
                'id': m.id,
                'title': m.title,
                'list': items,
                'depth': idx
              };
            }

            function process(m) {
              let items = [];
              for (const _m of m.values()) {
                let _g = convert(_m);
                if ( _g ){
                  items.push(_g);
                }
              }
              return items;
            }


            this.capabilityMap = process(map);

            let systems = new Map();

            for ( const c of result.capabilities ){
              if ( c.type === 'capability' && c.list ){
                for ( const s of c.list ){
                  let rec = systems.get(s.id);
                  if (!rec){
                    rec = {
                      title: s.title,
                      capabilities: new Set(),
                      products: new Set()
                    };
                    systems.set(s.id, rec);
                  }
                  rec.capabilities.add(c.id);
                }
              }
            }
            if ( result.products) {
              for (const p of result.products) {
                if (p.list) {
                  for (const s of p.list) {
                    let rec = systems.get(s.id);
                    if (!rec) {
                      rec = {
                        title: s.title,
                        capabilities: new Set(),
                        products: new Set()
                      };
                      systems.set(s.id, rec);
                    }
                    rec.products.add(p.id);
                  }
                }
              }
            }

            function lookup(selector){
              let caps = new Map();
              for ( const s of systems.keys() ){
                const _s = systems.get(s);
                if ( selector(_s)){
                  for ( const c of _s.capabilities){
                    let r = caps.get(c);
                    if (!r){
                      r = new Set();
                      caps.set(c, r);
                    }
                    r.add(s);
                  }
                }
              }
              let result = [];
              for ( const c of caps.keys()){
                const sys = [];
                for (const s of caps.get(c)){
                  sys.push({
                    'title': systems.get(s).title
                  });
                }
                result.push({
                  id: c,
                  systems: sys
                });
              }
              return result;
            }

            function products(products){
              let result = [];
              let out_scope = lookup((s)=>{
                return s.products.size === 0;
              });
              if ( out_scope.length > 0 ){
                result.push({
                  id: 'out.of.scope',
                  title: 'Вне продуктов',
                  type: 'no-scope',
                  capabilities: out_scope
                });
              }
              if ( products){
                for ( const p of products ){
                  result.push({
                    id: p.id,
                    title: p.title,
                    type: 'normal',
                    capabilities: lookup((s)=>{
                      return s.products.has(p.id);
                    })
                  });
                }
              }
              return result;
            }
            this.products = products(result.products);
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
.product-capability-header{
  min-height: 90px;
  width: 100%;
}
.product-capability-body{
  display: flex;
  width: 100%;
  flex-direction: column;
}

.product-capability-map {
  display: flex;
  width: 100%;
  flex-direction: column;
}

.product-list {
  position: absolute;
  display: flex;
  flex-direction: row;
  overflow-x: auto;
  overflow-y: hidden;
}

</style>
