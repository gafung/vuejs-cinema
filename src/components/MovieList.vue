<template>
    <div id="movie-list">
        <div v-if="filteredMovies.length">
            <movie-item v-for="movie in filteredMovies" v-bind:movie="movie.movie">
                <div class="movie-sessions">
                    <div
                            v-for="session in filterSessions(movie.sessions)"
                            class="session-time-wrapper tooltip-wrapper"
                            v-tooltip="{ seats: session.seats }"
                    >
                        <div class="session-time">{{ formatSessionTime(session.time) }}</div>
                    </div>
                </div>
            </movie-item>
        </div>
        <div v-else-if="movies.length">
            {{ noResult }}
        </div>
        <div v-else>
            Loading...
        </div>
    </div>
</template>

<script>
    import genres from '../util/genres';
    import MovieItem from './MovieItem.vue';
    import times from '../util/times';

    export default {
        methods: {
            formatSessionTime(raw) {
                return this.$moment(raw).format('h:mm A');
            },
            filterSessions(sessions){
                return sessions.filter(this.sessionPassesTimeFilter);
            },
            moviePassesGenreFilter(movie) {
                if(!this.genre.length){
                    return true;
                } else {
                    let movieGenres = movie.movie.Genre.split(', ');
                    let matched = true;
                    this.genre.forEach(genre=>{
                        if(movieGenres.indexOf(genre) === -1) {
                            matched = false;
                        }
                    });
                    return matched;
                }
            },
            sessionPassesTimeFilter(session) {
                if(!this.day.isSame(this.$moment(session.time), 'day')){
                    return false;
                }
                else if(this.time.length === 0 || this.time.length === 2){
                    return true;
                } else if(this.time[0] === times.AFTER_6PM){
                    return this.$moment(session.time).hour() >= 18;
                } else {
                    return this.$moment(session.time).hour() <= 18;
                }
            }
        },
        props: ['genre', 'time', 'movies', 'day'],
        computed: {
            filteredMovies() {
                return this.movies
                    .filter(this.moviePassesGenreFilter)
                    .filter(movie => {
                        return movie.sessions.find(this.sessionPassesTimeFilter);
                    });
            },
            noResult() {
                return `No results for ${this.genre.join(', ')}${this.genre.length && this.time.length ? ', ': ''}${this.time.join(', ')}`;
            }
        },
        components: {
            MovieItem
        }
    }
</script>