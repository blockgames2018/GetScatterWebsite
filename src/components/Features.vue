
<template>
    <section id="features" class="body">

        <div class="row">

          <Overview v-if="$route.params.feature === 'overview'" />
          <!-- <Stablecoins v-if="$route.params.feature === 'stablecoins'" /> -->
          <Assetmanagement v-if="$route.params.feature === 'assetmanagement'" />
          <Hardwarewallets v-if="$route.params.feature === 'hardwarewallets'" />
          <Tokenexchange v-if="$route.params.feature === 'tokenexchange'" />

          <!-- <section class="page">
              <component v-if="pageComponent" :is="pageComponent"></component>
          </section> -->

        </div>

    </section>

</template>

<script>

    import Overview from '../components/features/Overview';
    // import Stablecoins from '../components/features/Stablecoins';
    import Assetmanagement from '../components/features/Assetmanagement';
    import Hardwarewallets from '../components/features/Hardwarewallets';
    import Tokenexchange from '../components/features/Tokenexchange';

    const findRoute = (features, route) => {
        let res = null;
        if(!route) return null;
        features.map(feature => {
            if(!feature.hasOwnProperty('children')) return null;
            if(feature.route === route) res = feature;
            const found = findRoute(feature.children, route);
            if(found) return res = found;
        });
        return res;
    };

    const rebuildActiveLinks = (features, lastRoute) => {
        let links = [];
        let done = false;
        features.map(feature => {
            if(done) return false;
            if(feature.route === lastRoute) return links.push(lastRoute);

            if(feature.hasOwnProperty('children') && feature.children.length){
                const found = rebuildActiveLinks(feature.children, lastRoute);
                if(found.length){
                    links.push(feature.route);
                    links = links.concat(found);
                }
            }

            done = links.includes(lastRoute);
        });
        return links;
    };

    import {features} from '../data/features';

    export default {
        data(){ return {
            menu:features,
            activeLinks:[],
            mobileMenu:false,
        }},
        components:{
          Overview,
          // Stablecoins,
          Assetmanagement,
          Hardwarewallets,
          Tokenexchange
        },
        computed:{
            breadcrumbs(){
                let crumbs = [];
                this.activeLinks.reduce((lastObj,route) => {
                    if(lastObj === null) lastObj = this.menu.features.find(x => x.route === route);
                    else lastObj = lastObj.children.find(x => x.route === route)
                    crumbs.push(lastObj);
                    return lastObj;
                }, null);
                return crumbs.slice(crumbs.length - 2, crumbs.length);
            },
            pageComponent(){
                return this.activeLinks.length ? `${this.menu.base}-${this.activeLinks[this.activeLinks.length-1]}` : null;
            }
        },
        mounted(){
            console.log(features);
            this.activeLinks = rebuildActiveLinks(this.menu.features, this.$route.params.feature);
        },
        methods:{
            clickedMenuLink(feature){
                this.$router.push({name:'features', params:{type:this.menu.base, feature:feature.route}});
                if(!this.activeLinks.length) this.activeLinks.push(feature.route);
                else {
                    let index = 0;
                    const found = () => findRoute(this.menu.features, this.activeLinks[index]);
                    while(found()){
                        const possible = found();
                        if(!!possible && possible.hasOwnProperty('children') && possible.children.map(x => x.route).includes(feature.route)){
                            this.activeLinks = this.activeLinks.splice(0, index+1);
                            return this.activeLinks.push(feature.route);
                        }
                        index++;
                    }
                    this.activeLinks = [feature.route];
                }
            }
        },
        watch:{
            pageComponent(){
                window.scrollTo(0, 0)
            }
        }
    }
</script>

<style lang="scss">

  @import "../variables";

  #features {
      background-color:$white;
      text-align:left;

      .col-4 {
        margin:0 auto;
      }

      h2 {

      }

      .sidebar {
        width:100%;
        padding:12rem 3rem 3rem;

        figure.link {
          padding:1.4rem;
          border-radius:4px;
          cursor:pointer;
          background-color:white;
          transition:background-color 0.18s ease-in-out;

          &.active {
            color:$blue;
            font-weight:bold;
            background-color:$background;
          }
        }
      }
  }

</style>