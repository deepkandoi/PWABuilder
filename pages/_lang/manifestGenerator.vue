<template>
<section>
  <GeneratorMenu :first-link-path="true" />
  <div class="l-generator-step">
    <div class="pure-g l-generator-padded">
      <div class="pure-u-1 pure-u-md-3-5 pure-u-lg-2-5">
        <header class="l-generator-header l-generator-header--minimal">
          <h2 class="l-generator-title">{{ $t('generator.title') }}</h2>
          <h4 class="l-generator-subtitle">
            {{ $t('generator.subtitle') }}
          </h4>
        </header>

        <div class="l-generator-form">
          <form @submit.prevent="checkUrlAndGenerate" @keydown.enter.prevent="checkUrlAndGenerate">
            <div class="l-generator-field">
              <label class="l-generator-label" for="siteUrl">{{ $t('generator.url') }}</label>
              <input class="l-generator-input" :placeholder="$t('generator.placeholder_url')" name="siteUrl" id="siteUrl" type="text" ref="url"
                v-model="url$" autofocus>
            </div>

            <div class="pure-g l-breath">
              <div class="l-generator-wrapper pure-u-3-5">
                <button type="submit" class="get-started pwa-button isEnabled next-step" @click=" $awa( { 'referrerUri': 'https://www.pwabuilder.com/build/manifest-scan' })">
                  {{ $t('generator.start') }}
                  <Loading :active="inProgress" class="u-display-inline_block u-margin-left-sm"/>
                </button>
              </div>

              <div class="pure-u-2-5">
              </div>

              <div class="pure-u-1">
                <p class="l-generator-error" v-if="error">
                  <span class="icon-exclamation"></span>
                  {{ $t(error) }}
                </p>
              </div>

              <div class="l-generator-wrapper pure-u-1">
                <button @click="skipCheckUrl(); $awa( { 'referrerUri': 'https://www.pwabuilder.com/skip/service-worker' })"
                  class="pwa-button pwa-button--simple">
                  {{ $t('generator.skip') }}
                </button>
              </div>
            </div>
            <p class="l-narrow">{{ $t('generator.skip_description') }}</p>
          </form>
        </div>
      </div>
    </div>
  </div>
</section>
</template>

<script lang='ts'>
import Vue from 'vue';
import Component from 'nuxt-class-component';
import { Action, State, namespace } from 'vuex-class';

import GeneratorMenu from '~/components/GeneratorMenu.vue';
import Loading from '~/components/Loading.vue';

import * as generator from '~/store/modules/generator';

const GeneratorState = namespace(generator.name, State);
const GeneratorAction = namespace(generator.name, Action);

@Component({
  components: {
    GeneratorMenu,
    Loading
  }
})
export default class extends Vue {
  public url$: string | null = null;
  public generatorReady = true;
  public error: string | null = null;

  @GeneratorState url: string;
  @GeneratorAction updateLink;
  @GeneratorAction getManifestInformation;

  public created(): void {
    this.url$ = this.url;
  }

  public mounted(): void {
    const overrideValues = {
      isAuto: false,
      behavior: 0,
      uri: window.location.href,
      pageName: "manifestGenerator",
      pageHeight: window.innerHeight
    };

    if (awa) {
      awa.ct.capturePageView(overrideValues);
    }
  }

  public get inProgress(): boolean {
    return !this.generatorReady && !this.error;
  }

  public skipCheckUrl(): void {
    this.$router.push({
      name: 'serviceworker'
    });
  }

  public async checkUrlAndGenerate(): Promise<void> {
    this.generatorReady = false;
    this.error = null;

    try {
      this.updateLink(this.url$);

      if (!this.url$) {
        return;
      }

      this.url$ = this.url;
      await this.getManifestInformation();

      this.$router.push({
        name: 'generate'
      });
    } catch (e) {
      this.error = e;
    }
  }
}

declare var awa: any;

</script>
