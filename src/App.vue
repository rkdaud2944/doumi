<template>
    <NavMenu />
    <FrameWrapper />

    <SignUp />

</template>

<script>
import NavMenu from './components/NavMenu.vue';
import FrameWrapper from './components/FrameWrapper.vue';
import SignUp from './components/SignUp.vue';

export default {
  name: 'App',
  components: {
    NavMenu,
    FrameWrapper,
    SignUp
  }
};
</script>
