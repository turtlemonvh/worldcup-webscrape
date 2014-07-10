# Grabbing world cup data to visualize

I saw the heat map on the player stats page, noticed it wasn't rendering well on FireFox, so I started wondering what data I could get off to play with myself.

Note that [they dont want robots](http://www.fifa.com/robots.txt), and [the "Content" section of their TOS says you can't reproduce their data](http://www.fifa.com/legal/tos.html), so be gentle with the scraping.

I'm not grabbing anything off of there yet because I don't have anything interesting to do with it right now.

## All player data

I got this for you. From http://www.fifa.com/worldcup/players/browser/index.html.  It's in the `script` container with `id = playerJSon`.

The data I'd pull out of this:

* player ids
* team id
* team name

## Matches

Match list from: http://www.fifa.com/worldcup/matches/index.html

Each match is an element on the page following this form:

        <div class="col-xs-12 clear-grid ">
            <div data-id="300186504" class="mu result">
                <a href="/worldcup/matches/round=255953/match=300186504/index.html#nosticky" class="mu-m-link">
                    <div class="mu-i">
                        <div class="mu-i-datetime">05 Jul 2014 - 13:00
                            <span class="wrap-localtime">Local time</span>
                        </div>
                        <div class="mu-i-date">05 Jul 2014</div>
                        <div class="mu-i-matchnum">Match 60</div>
                        <div class="mu-i-group">Quarter-finals</div>
                        <div class="mu-i-location">
                            <div class="mu-i-stadium">Estadio Nacional</div>
                            <div class="mu-i-venue">Brasilia </div>
                        </div>
                        </div>
                        <div class="mu-day">
                            <span class="t-day">05</span>
                            <span class="t-month">Jul</span>
                        </div>
                        <div class="mu-m">
                            <div class="t home" data-team-id="43922">
                                <div class="t-i i-4">
                                    <span class="t-i-wrap">
                                        <img src="http://img.fifa.com/images/flags/4/arg.png" alt="Argentina" class="ARG i-4-flag flag" />
                                    </span>
                                </div>
                                <div class="t-n">
                                    <span class="t-nText ">Argentina</span>
                                    <span class="t-nTri">ARG</span>
                                </div>
                            </div>
                            <div class="t away" data-team-id="43935">
                                <div class="t-i i-4">
                                    <span class="t-i-wrap">
                                        <img src="http://img.fifa.com/images/flags/4/bel.png" alt="Belgium" class="BEL i-4-flag flag" />
                                    </span>
                                </div>
                                <div class="t-n">
                                    <span class="t-nText ">Belgium</span>
                                    <span class="t-nTri">BEL</span>
                                </div>
                            </div>
                            <div class="s">
                                <div class="s-fixture">
                                    <div class="s-status">Full-time </div>
                                    <div class="s-status-abbr">FT </div>
                                    <div class="s-score s-date-HHmm" data-daymonthutc="0507">
                                    <span class="s-scoreText">1-0</span>
                                </div>
                            </div>
                        </div>
                        <div class="mu-reasonwin">
                            <span class="text-reasonwin"></span>
                            <span class="icon-mrep"> </span>
                            <span class="icon-hl"> </span>
                        </div><div class="mu-reasonwin-abbr">
                        <span class="text-reasonwin"> </span>
                        <span class="icon-mrep"> </span>
                        <span class="icon-hl"> </span>
                        </div>
                    </div>
                </a>
            </div>
         </div>

         
### Grabbing data off this page

    // Number of matches
    Grp Rnds: 8 groups * 6 matches per group = 48
    Rnd 16: 8 games
    Rnd 8 (quarter): 4 games
    Rnd 4 (semi): 2 games
    Final: 2 games (final + 3rd place)
    ++++++++++++
    Total = 48+8+4+2+2 = 64 games

    // Grabbing them off the page
    // First 3 of each set are repeats
    $('.clear-grid .mu.result').length // 65 - 3 = 62 played
    $('.clear-grid .mu.fixture').length // 5 - 3 = 2 upcoming (final and play off)

From this you can get the data you need:

* Team ids
* All the match ids
* All the round ids

Like this

    var t = [];
    var rg = new RegExp("fifa.com/worldcup/matches/round=(.*)/match=(.*)/index.html")
    $.each( $('.clear-grid .mu'), function(i, d){ $d = $(d); mtch=$d.find('.mu-m-link')[0].href.match(rg); tms=$d.find('div[data-team-id]'); tmns=tms.find('.t-nText'); t.push({ round: mtch[1], match: mtch[2], teams: [{id: tms[0].getAttribute('data-team-id'), name: tmns[0].innerHTML}, {id: tms[1].getAttribute('data-team-id'), name: tmns[1].innerHTML}] }); })
    JSON.stringify(t) // spits out the data you want

The result is in the file `matches.json`, which has 70 entries.  I didn't get rid of the repeats yet.

## Grabbing matches per player 

For each player, grab their matches with: http://www.fifa.com/worldcup/players/player=336085/played-matches.html.  Grab the contents of these option blocks:

    // E.g.
    $.each($('.player-match-select option'), function(i, v){ console.log(v.value) })

The first part is the round id, and the second is the match id.  I didn't get this data.

## Grabbing stats per player per match

Make queries to urls that look like this:

    http://lup.fifa.com/live/common/competitions/worldcup/round=255953/match=300186488/statistics/player=336098/player.js

You'll have to clean up the data, since its not json but a js callback function.

That's it.  Hopefully FIFA is cool with this.