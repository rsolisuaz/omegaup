<template>
  <div class="mt-2">
    <div class="table-responsive">
      <table
        class="runs table table-striped"
        :class="{ global: showAllRuns, local: !showAllRuns }"
      >
        <caption>
          {{
            T.wordsSubmissions
          }}
          <div v-if="showPager">
            <button :disabled="filterOffset <= 0" @click="filterOffset--">
              &lt;
            </button>
            {{ filterOffset + 1 }}
            <button :disabled="runs.length < rowCount" @click="filterOffset++">
              &gt;
            </button>

            <label
              >{{ T.wordsVerdict }}:
              <select v-model="filterVerdict" class="form-control">
                <option value="">{{ T.wordsAll }}</option>
                <option value="AC">AC</option>
                <option value="PA">PA</option>
                <option value="WA">WA</option>
                <option value="TLE">TLE</option>
                <option value="MLE">MLE</option>
                <option value="OLE">OLE</option>
                <option value="RTE">RTE</option>
                <option value="RFE">RFE</option>
                <option value="CE">CE</option>
                <option value="JE">JE</option>
                <option value="VE">VE</option>
                <option value="NO-AC">No AC</option>
              </select>
            </label>

            <label
              >{{ T.wordsStatus }}:
              <select v-model="filterStatus" class="form-control">
                <option value="">{{ T.wordsAll }}</option>
                <option value="new">new</option>
                <option value="waiting">waiting</option>
                <option value="compiling">compiling</option>
                <option value="running">running</option>
                <option value="ready">ready</option>
              </select>
            </label>

            <label
              >{{ T.wordsLanguage }}:
              <select v-model="filterLanguage" class="form-control">
                <option value="">{{ T.wordsAll }}</option>
                <option value="cpp17-gcc">C++17 (g++ 9.3)</option>
                <option value="cpp17-clang">C++17 (clang++ 10.0)</option>
                <option value="cpp11-gcc">C++11 (g++ 9.3)</option>
                <option value="cpp11-clang">C++11 (clang++ 10.0)</option>
                <option value="c11-gcc">C (gcc 9.3)</option>
                <option value="c11-clang">C (clang 10.0)</option>
                <option value="cs">C# (8.0, dotnet 3.1)</option>
                <option value="hs">Haskell (ghc 8.6)</option>
                <option value="java">Java (openjdk 14.0)</option>
                <option value="pas">Pascal (fpc 3.0)</option>
                <option value="py3">Python 3.8</option>
                <option value="py2">Python 2.7</option>
                <option value="rb">Ruby (2.7)</option>
                <option value="lua">Lua (5.3)</option>
                <option value="kp">Karel (Pascal)</option>
                <option value="kj">Karel (Java)</option>
                <option value="cat">{{ T.wordsJustOutput }}</option>
              </select>
            </label>

            <template v-if="showProblem">
              <label
                >{{ T.wordsProblem }}:
                <omegaup-autocomplete
                  v-model="filterProblem"
                  :init="initProblemAutocomplete"
                ></omegaup-autocomplete>
              </label>
              <button
                type="button"
                class="close"
                style="float: none"
                @click="filterProblem = ''"
              >
                &times;
              </button>
            </template>

            <template v-if="showUser">
              <label
                >{{ T.wordsUser }}:
                <omegaup-autocomplete
                  v-model="filterUsername"
                  :init="initUserAutocomplete"
                ></omegaup-autocomplete>
              </label>
              <button
                type="button"
                class="close"
                style="float: none"
                @click="filterUsername = ''"
              >
                &times;
              </button>
            </template>

            <div class="row">
              <div v-if="filters.length > 0" class="col-sm col-12">
                <span
                  v-for="filter in filters"
                  :key="filter.name"
                  class="btn btn-secondary mr-3"
                >
                  <span class="mr-2"
                    >{{ filter.name }}: {{ filter.value }}</span
                  >
                  <a @click="onRemoveFilter(filter.name)">
                    <font-awesome-icon :icon="['fas', 'times']" />
                  </a>
                </span>
                <a href="#" @click="onRemoveFilter('all')">
                  <span class="mr-2">{{ T.wordsRemoveFilter }}</span>
                </a>
              </div>
            </div>
          </div>
        </caption>
        <thead>
          <tr>
            <th>{{ T.wordsTime }}</th>
            <th>GUID</th>
            <th v-if="showUser">{{ T.wordsUser }}</th>
            <th v-if="showContest">{{ T.wordsContest }}</th>
            <th v-if="showProblem">{{ T.wordsProblem }}</th>
            <th>{{ T.wordsStatus }}</th>
            <th v-if="showPoints" class="numeric">{{ T.wordsPoints }}</th>
            <th v-if="showPoints" class="numeric">{{ T.wordsPenalty }}</th>
            <th v-if="!showPoints" class="numeric">{{ T.wordsPercentage }}</th>
            <th>{{ T.wordsLanguage }}</th>
            <th class="numeric">{{ T.wordsMemory }}</th>
            <th class="numeric">{{ T.wordsRuntime }}</th>
            <th v-if="showDetails && !showDisqualify && !showRejudge">
              {{ T.wordsActions }}
            </th>
            <th v-else></th>
          </tr>
        </thead>
        <tfoot v-if="problemAlias != null">
          <tr>
            <td colspan="10">
              <a
                v-if="isContestFinished"
                :href="`/arena/${contestAlias}/practice/`"
                >{{ T.arenaContestEndedUsePractice }}</a
              >
              <a
                v-else
                :href="newSubmissionUrl"
                @click="$emit('new-submission')"
                >{{ newSubmissionDescription }}</a
              >
            </td>
          </tr>
        </tfoot>
        <tbody>
          <tr v-for="run in filteredRuns" :key="run.guid">
            <td>{{ time.formatTimestamp(run.time) }}</td>
            <td>
              <acronym :title="run.guid" data-run-guid>
                <tt>{{ run.guid.substring(0, 8) }}</tt>
              </acronym>
            </td>
            <td v-if="showUser" class="text-break-all">
              <omegaup-user-username
                :classname="run.classname"
                :username="run.username"
                :country="run.country_id"
                :linkify="true"
                :emit-click-event="true"
                @click="(username) => (filterUsername = username)"
              ></omegaup-user-username>
              <a :href="`/profile/${run.username}/`" class="ml-2">
                <font-awesome-icon :icon="['fas', 'external-link-alt']" />
              </a>
            </td>
            <td v-if="showContest" class="text-break-all">
              <a
                href="#"
                @click="onEmitFilterChanged(run.contest_alias, 'contest')"
                >{{ run.contest_alias }}</a
              >
              <a
                v-if="run.contest_alias"
                :href="`/arena/${run.contest_alias}/`"
                class="ml-2"
              >
                <font-awesome-icon :icon="['fas', 'external-link-alt']" />
              </a>
            </td>
            <td v-if="showProblem" class="text-break-all">
              <a href="#" @click.prevent="filterProblem = run.alias">{{
                run.alias
              }}</a>
              <a :href="`/arena/problem/${run.alias}/`" class="ml-2">
                <font-awesome-icon :icon="['fas', 'external-link-alt']" />
              </a>
            </td>
            <td
              :style="{ backgroundColor: statusColor(run) }"
              data-run-status
              class="text-center"
            >
              <span>{{ status(run) }}</span>

              <button
                v-if="!!statusHelp(run)"
                type="button"
                :data-content="statusHelp(run)"
                data-toggle="popover"
                data-trigger="focus"
                class="btn btn-outline-dark btn-sm"
                @click="showVerdictHelp"
              >
                <font-awesome-icon :icon="['fas', 'question-circle']" />
              </button>
            </td>
            <td v-if="showPoints" class="numeric">{{ points(run) }}</td>
            <td v-if="showPoints" class="numeric">{{ penalty(run) }}</td>
            <td v-if="!showPoints" class="numeric">{{ percentage(run) }}</td>
            <td>{{ run.language }}</td>
            <td class="numeric">{{ memory(run) }}</td>
            <td class="numeric">{{ runtime(run) }}</td>
            <td v-if="showDetails && !showDisqualify && !showRejudge">
              <button
                class="details btn btn-outline-dark btn-sm"
                data-run-details
                @click="$emit('details', run)"
              >
                <font-awesome-icon :icon="['fas', 'search-plus']" />
              </button>
            </td>
            <td v-else-if="showDetails || showDisqualify || showRejudge">
              <div class="dropdown">
                <button
                  class="btn btn-secondary dropdown-toggle"
                  type="button"
                  data-toggle="dropdown"
                  aria-haspopup="true"
                  aria-expanded="false"
                >
                  {{ T.wordsActions }}
                  <span class="caret"></span>
                </button>
                <ul class="dropdown-menu">
                  <li v-if="showDetails" data-actions-details>
                    <button
                      class="btn btn-link dropdown-item"
                      @click="$emit('details', run)"
                    >
                      {{ T.wordsDetails }}
                    </button>
                  </li>
                  <li v-if="showRejudge" data-actions-rejudge>
                    <button
                      class="btn btn-link dropdown-item"
                      @click="$emit('rejudge', run)"
                    >
                      {{ T.wordsRejudge }}
                    </button>
                  </li>
                  <li role="separator" class="divider"></li>
                  <li v-if="showDisqualify" data-actions-disqualify>
                    <button
                      class="btn btn-link dropdown-item"
                      @click="$emit('disqualify', run)"
                    >
                      {{ T.wordsDisqualify }}
                    </button>
                  </li>
                </ul>
              </div>
            </td>
            <td v-else></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch, Emit } from 'vue-property-decorator';
import T from '../../lang';
import { types } from '../../api_types';
import * as time from '../../time';
import * as typeahead from '../../typeahead';
import user_Username from '../user/Username.vue';

import Autocomplete from '../Autocomplete.vue';

import { library } from '@fortawesome/fontawesome-svg-core';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import {
  faQuestionCircle,
  faRedoAlt,
  faBan,
  faSearchPlus,
  faExternalLinkAlt,
  faTimes,
} from '@fortawesome/free-solid-svg-icons';
library.add(faQuestionCircle);
library.add(faRedoAlt);
library.add(faBan);
library.add(faSearchPlus);
library.add(faExternalLinkAlt);
library.add(faTimes);

declare global {
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  interface JQuery {
    popover(action: string): JQuery;
  }
}

@Component({
  components: {
    FontAwesomeIcon,
    'omegaup-autocomplete': Autocomplete,
    'omegaup-user-username': user_Username,
  },
})
export default class Runs extends Vue {
  @Prop({ default: false }) showAllRuns!: boolean;
  @Prop({ default: false }) isContestFinished!: boolean;
  @Prop({ default: true }) isProblemsetOpened!: boolean;
  @Prop({ default: false }) showContest!: boolean;
  @Prop({ default: false }) showDetails!: boolean;
  @Prop({ default: false }) showDisqualify!: boolean;
  @Prop({ default: false }) showPager!: boolean;
  @Prop({ default: false }) showPoints!: boolean;
  @Prop({ default: false }) showProblem!: boolean;
  @Prop({ default: false }) showRejudge!: boolean;
  @Prop({ default: false }) showUser!: boolean;
  @Prop({ default: null }) contestAlias!: string | null;
  @Prop({ default: null }) problemAlias!: string | null;
  @Prop({ default: null }) problemsetProblems!: types.ProblemsetProblem[];
  @Prop({ default: null }) username!: string | null;
  @Prop({ default: 100 }) rowCount!: number;
  @Prop() runs!: types.Run[];

  T = T;
  time = time;
  typeahead = typeahead;

  filterLanguage: string = '';
  filterOffset: number = 0;
  filterProblem: string = '';
  filterStatus: string = '';
  filterUsername: string = '';
  filterVerdict: string = '';
  filterContest: string = '';
  filters: { name: string; value: string }[] = [];

  get filteredRuns(): types.Run[] {
    if (
      !this.filterLanguage &&
      !this.filterProblem &&
      !this.filterStatus &&
      !this.filterUsername &&
      !this.filterContest &&
      !this.filterVerdict
    ) {
      return this.sortedRuns;
    }
    return this.sortedRuns.filter((run) => {
      if (this.filterVerdict) {
        if (this.filterVerdict == 'NO-AC') {
          if (run.verdict == 'AC') {
            return false;
          }
        } else if (run.verdict != this.filterVerdict) {
          return false;
        }
      }
      if (this.filterLanguage && run.language !== this.filterLanguage) {
        return false;
      }
      if (this.filterProblem && run.alias !== this.filterProblem) {
        return false;
      }
      if (this.filterStatus && run.status !== this.filterStatus) {
        return false;
      }
      if (this.filterUsername && run.username !== this.filterUsername) {
        return false;
      }
      if (this.filterContest && run.contest_alias !== this.filterContest) {
        return false;
      }
      return true;
    });
  }

  get sortedRuns(): types.Run[] {
    return this.runs
      .slice()
      .sort((a, b) => b.time.getTime() - a.time.getTime());
  }

  get newSubmissionUrl(): string {
    if (this.isProblemsetOpened) {
      return `#problems/${this.problemAlias}/new-run`;
    }
    return `/arena/${this.contestAlias}/`;
  }

  get newSubmissionDescription(): string {
    if (this.isProblemsetOpened) {
      return T.wordsNewSubmissions;
    }
    return T.arenaContestNotOpened;
  }

  // eslint-disable-next-line no-undef -- This is defined in TypeScript.
  initProblemAutocomplete(el: JQuery<HTMLElement>) {
    if (this.problemsetProblems.length !== 0) {
      typeahead.problemsetProblemTypeahead(
        el,
        () => this.problemsetProblems,
        (event: Event, item: { alias: string; title: string }) => {
          this.filterProblem = item.alias;
        },
      );
    } else {
      typeahead.problemTypeahead(el);
    }
  }

  // eslint-disable-next-line no-undef -- This is defined in TypeScript.
  initUserAutocomplete(el: JQuery<HTMLElement>) {
    if (this.problemsetProblems.length !== 0 && this.contestAlias) {
      typeahead.userContestTypeahead(el, this.contestAlias);
    } else {
      typeahead.userTypeahead(el);
    }
  }

  memory(run: types.Run): string {
    if (
      run.status == 'ready' &&
      run.verdict != 'JE' &&
      run.verdict != 'VE' &&
      run.verdict != 'CE'
    ) {
      let prefix = '';
      if (run.verdict == 'MLE') {
        prefix = '>';
      }
      return `${prefix}${(run.memory / (1024 * 1024)).toFixed(2)} MB`;
    } else {
      return '—';
    }
  }

  penalty(run: types.Run): string {
    if (
      run.status == 'ready' &&
      run.verdict != 'JE' &&
      run.verdict != 'VE' &&
      run.verdict != 'CE'
    ) {
      return run.penalty.toFixed(2);
    }
    return '—';
  }

  percentage(run: types.Run): string {
    if (
      run.status == 'ready' &&
      run.verdict != 'JE' &&
      run.verdict != 'VE' &&
      run.verdict != 'CE'
    ) {
      return `${(run.score * 100).toFixed(2)}%`;
    }
    return '—';
  }

  points(run: types.Run): string {
    if (
      run.status == 'ready' &&
      run.verdict != 'JE' &&
      run.verdict != 'VE' &&
      run.verdict != 'CE' &&
      typeof run.contest_score !== 'undefined'
    ) {
      return run.contest_score.toFixed(2);
    }
    return '—';
  }

  runtime(run: types.Run): string {
    if (
      run.status == 'ready' &&
      run.verdict != 'JE' &&
      run.verdict != 'VE' &&
      run.verdict != 'CE'
    ) {
      let prefix = '';
      if (run.verdict == 'TLE') {
        prefix = '>';
      }
      return `${prefix}${(run.runtime / 1000).toFixed(2)} s`;
    }
    return '—';
  }

  showVerdictHelp(ev: Event): void {
    $(ev.target as HTMLElement).popover('show');
  }

  statusColor(run: types.Run): string {
    if (run.status != 'ready') return '';

    if (run.type == 'disqualified') return '#F00';

    if (run.verdict == 'AC') {
      return '#CF6';
    } else if (run.verdict == 'CE') {
      return '#F90';
    } else if (run.verdict == 'JE' || run.verdict == 'VE') {
      return '#F00';
    } else {
      return '';
    }
  }

  status(run: types.Run): string {
    if (run.type == 'disqualified') return T.wordsDisqualified;

    return run.status == 'ready' ? run.verdict : run.status;
  }

  statusHelp(run: types.Run): string {
    if (run.status != 'ready' || run.verdict == 'AC') {
      return '';
    }

    if (run.language == 'kj' || run.language == 'kp') {
      if (run.verdict == 'RTE' || run.verdict == 'RE') {
        return T.verdictHelpKarelRTE;
      } else if (run.verdict == 'TLE' || run.verdict == 'TO') {
        return T.verdictHelpKarelTLE;
      }
    }
    if (run.type == 'disqualified') return T.verdictHelpDisqualified;
    const verdict = T[`verdict${run.verdict}`];
    const verdictHelp = T[`verdictHelp${run.verdict}`];

    return `${verdict}: ${verdictHelp}`;
  }

  @Watch('username')
  onUsernameChanged(newValue: string | null) {
    if (!newValue) {
      this.filterUsername = '';
    } else {
      this.filterUsername = newValue;
    }
  }

  @Watch('problemAlias')
  onProblemAliasChanged(newValue: string | null) {
    if (!newValue) {
      this.filterProblem = '';
    } else {
      this.filterProblem = newValue;
    }
  }

  @Watch('filterLanguage')
  onFilterLanguageChanged(newValue: string) {
    this.onEmitFilterChanged(newValue, 'language');
  }

  @Watch('filterOffset')
  onFilterOffsetChanged() {
    this.$emit('filter-changed');
  }

  @Watch('filterProblem')
  onFilterProblemChanged(newValue: string) {
    this.onEmitFilterChanged(newValue, 'problem');
  }

  @Watch('filterStatus')
  onFilterStatusChanged(newValue: string) {
    this.onEmitFilterChanged(newValue, 'status');
  }

  @Watch('filterUsername')
  onFilterUsernameChanged(newValue: string) {
    this.onEmitFilterChanged(newValue, 'username');
  }

  @Watch('filterVerdict')
  onFilterVerdictChanged(newValue: string) {
    this.onEmitFilterChanged(newValue, 'verdict');
  }

  @Emit('filter-changed')
  onEmitFilterChanged(value: string, filter: string): void {
    this.filterOffset = 0;
    if (!value) {
      this.filters = this.filters.filter((item) => item.name !== filter);
      return;
    }
    if (filter === 'contest') {
      // This field does not appear as filter
      this.filterContest = value;
    }
    const currentFilter = this.filters.find((item) => item.name === filter);
    if (!currentFilter) {
      this.filters.push({ name: filter, value: value });
    } else {
      currentFilter.value = value;
    }
  }

  onRemoveFilter(filter: string): void {
    if (filter === 'all') {
      this.filterLanguage = '';
      this.filterProblem = '';
      this.filterStatus = '';
      this.filterUsername = '';
      this.filterVerdict = '';
      this.filterContest = '';
      this.filterOffset = 0;

      this.filters = [];
      return;
    }
    switch (filter) {
      case 'language':
        this.filterLanguage = '';
        break;
      case 'problem':
        this.filterProblem = '';
        break;
      case 'status':
        this.filterStatus = '';
        break;
      case 'username':
        this.filterUsername = '';
        break;
      case 'verdict':
        this.filterVerdict = '';
        break;
      case 'contest':
        this.filterContest = '';
    }
    this.filters = this.filters.filter((item) => item.name !== filter);
  }
}
</script>

<style lang="scss" scoped>
caption {
  caption-side: top;
}

.text-break-all {
  word-break: break-all;
}

.runs {
  width: 100%;
  border: 1px solid #ccc;
  margin-top: 2em;
}

.runs caption {
  font-weight: bold;
  font-size: 1em;
  margin-bottom: 1em;
}

.runs td,
.runs th {
  border: 1px solid #ccc;
  border-width: 1px 0;
  text-align: center;
}

.runs tfoot td a {
  display: block;
  padding: 0.5em;
  text-decoration: none;
  color: #000;
  background: #ccc;
  text-align: center;
}

.runs tfoot td a:hover {
  background: #fff;
}
</style>
