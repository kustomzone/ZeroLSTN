<template>
    <div id="Music">
        <div class="row">
            <div class="col s12">
                <ul class="tabs tabs-fixed-width">
                    <li v-for="tab in tabs" class="tab col s3">
                        <a :href="'#' + tab.show">{{ tab.name }}</a>
                    </li>
                </ul>
            </div>
            <div id="artists" class="col s12">
                <ul class="collection with-header">
                    <li class="collection-header"><h4>Artists</h4></li>
                    <li v-for="artist in artists" class="collection-item">
                        <a href="#!" @click.prevent="goToArtist(artist)">{{ artist }}</a>
                    </li>
                </ul>
            </div>
            <div id="albums" class="col s12">
                <ul class="collection with-header">
                    <li class="collection-header"><h4>Albums</h4></li>
                    <li v-for="album in albums" class="collection-item">
                        <a href="#!" @click.prevent="goToAlbum(album)">{{ album }}</a>
                    </li>
                </ul>
            </div>
            <div id="songs" class="col s12">
                <ul class="collection with-header">
                    <li class="collection-header"><h4>Songs</h4></li>
                    <songitem  v-for="song in songs" :editable="false" :song="song"></songitem>
                </ul>
            </div>
            <div id="genres" class="col s12">
                <div class="row"></div>
                <div class="row">
                    <div class="col s12">
                        <a @click.prevent="addNewGenre()" class="btn waves-effect waves-light right"><i class="material-icons left">add</i>Add Genre</a>
                    </div>
                </div>
                <ul class="collection with-header">
                    <li class="collection-header"><h4>Genres</h4></li>
                    <li v-for="genre in connectedGenres" class="collection-item">
                        {{ genre.name }}
                    </li>
                    <li v-for="genre in genres" class="collection-item">
                        {{ genre.name }}
                        <a class="secondary-content">
                            <a href="#" @click.prevent="addGenre(genre.address)"><i class="material-icons">add</i></a>
                        </a>
                    </li>
                </ul>
            </div>
            <!--
            <div id="playlists" class="col s12">
                <ul class="collapsible" data-collapsible="expandable">
                    <li>
                    <div class="collapsible-header"><i class="material-icons">playlist_play</i>Best Songs 2017</div>
                    <div class="collapsible-body">
                        <div class="row">
                        <div class="col s12">
                            <ul class="collection with-header">
                                <li class="collection-header"><h4>Best Songs 2017</h4></li>
                                <li class="collection-item">Strawberry Swing</li>
                                <li class="collection-item">Past Winters</li>
                                <li class="collection-item">Snowcone</li>
                                <li class="collection-item">Here We Play</li>
                            </ul>
                        </div>
                        </div>
                    </div>
                    </li>
                    <li>
                    <div class="collapsible-header"><i class="material-icons">playlist_play</i>Second</div>
                    <div class="collapsible-body"><span>Lorem ipsum dolor sit amet.</span></div>
                    </li>
                    <li>
                    <div class="collapsible-header"><i class="material-icons">playlist_play</i>Third</div>
                    <div class="collapsible-body"><span>Lorem ipsum dolor sit amet.</span></div>
                    </li>
                </ul>
            </div>
            -->
        </div>
        <div class="row"></div>
        <div class="row"></div>
        <div class="row"></div>
    </div>
</template>

<script>
    var Router = require("../libs/router.js");
    var SongItem = require("../vue_components/song_item.vue");

    module.exports = {
        components: {
            songitem: SongItem
        },
        props: [],
        name: "Music",
        beforeMount: function() {
            // TODO: Paginate

            // Get all known artists
            var self = this;
            page.getAllArtists(limit=20, offset=0)
                .then((artists) => {
                    self.artists = artists;
                });

            // Get all known albums
            page.getAllAlbums(limit=20, offset=0)
                .then((albums) => {
                    self.albums = albums;
                });

            // Get all known songs
            page.getAllSongs(limit=20, offset=0)
                .then((songs) => {
                    self.songs = songs;
                });

            // Get hardcoded genres/merger zites
            page.getGenresFromIndex()
                .then((genres) => {
                    self.genres = genres;
                });

            // Get genres we've already added
            // TODO: Make this more efficient
            page.getConnectedGenres()
                .then((connectedGenres) => {
                    self.connectedGenres = connectedGenres;
                    var newGenres = [];
                    self.genres.forEach(genre => { // For each hardcoded genre
                        // Check if we're already connected
                        var shouldAdd = true;
                        
                        // If we are, delete it from the hardcoded list.
                        // Any genre on the hardcoded list will show up with a '+' to add it
                        // Afterwards, any genres in connectedGenres will be shown with no '+'
                        for (var i = 0; i < connectedGenres.length; i++) {
                            if (genre.address === connectedGenres[i].address) {
                                shouldAdd = false;
                            } else if (connectedGenres[i].address === "1GEnReVHyvRwC4BR32UnVwHX7npUmxVpiY" ||
                              connectedGenres[i].address === "1iNdEXm7ZNDpwyHHTtsh7QMiMDyx2wUZB") {
                                // Remove default genre site and index from merger site results
                                self.connectedGenres.splice(i, 1);
                            }
                            
                        }
                        if (shouldAdd) {
                            newGenres.push(genre);
                        }
                    });            
                    self.genres = newGenres;
                });
        },
        mounted: function() {
            // Initialize tabs
            var tabs = document.querySelector("ul.tabs");
            var instance = new M.Tabs(tabs, {});

            // Initialize collapsible
            // TODO: Uncomment when playlists are live again
            //var collap = document.querySelector("ul.collapsible");
            //var collapInstance = new M.Collapsible(collap, {});
        },
        data: () => {
            return {
                tabs: [
                    { name: "Artists", icon: "people", active: false, show: "artists" },
                    { name: "Albums", icon: "album", active: false, show: "albums" },
                    { name: "Songs", icon: "music_note", active: true, show: "songs" },
                    { name: "Genres", icon: "library_music", active: false, show: "genres" }
                    //{ name: "Playlists", icon: "playlist_play", active: false, show: "playlists" }
                ],
                artists: [],
                albums: [],
                genres: [],
                connectedGenres: [],
                songs: [],
                playlists: [],
                genreModal: null
            }
        },
        methods: {
            goToArtist: function(artist) {
                // Tell the parent view to go to the specified artist's page
                this.$parent.goToArtistPage(artist)
            },
            goToAlbum: function(album) {
                // Tell the parent view to go to the specified album's page
                this.$parent.goToAlbumPage(album)
            },
            addGenre: function(address) {
                // Add a genre by clicking the '+' on the genre page
                page.addMerger(address);
            },
            addNewGenre: function() {
                // Clone and navigate to the default genre site
                page.cmdp("siteClone", ["1GEnReVHyvRwC4BR32UnVwHX7npUmxVpiY"]);
            }
        }
    }
</script>