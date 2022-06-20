<template>
<header>
     <img class="pokemon" src="../../assets/img/pokemont.png"/>
</header>

</template>

<script>
    export default{
        name: 'TheHeader'
    }
</script>

<style scoped>
    header{
        background: #005a6b;
        color: #fff;
        padding: .5rem;
        text-align: center;
    }
    h1{
        font-size: 2rem;
        font-weight: 600;
    }
    .pokemon{
        height: 80px;
        width: auto;
    }
</style>