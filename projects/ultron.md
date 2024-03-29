---
title: Ultron
github: pikesley/ultron
tags: ["ruby"]
layout: page
---
Ruby bindings for the [Marvel Comics API](http://developer.marvel.com/)

    git clone https://github.com/pikesley/ultron
    cd ultron
    bundle
    rake

Or just

    gem install ultron

To actually use it you need an API keypair which you can get from [here](https://developer.marvel.com/signup), put them into `~/.ultronrc` or `.env` like this

    PUBLIC_KEY: this_r_public_key
    PRIVATE_KEY: this_one_r_private_key

I've built [another gem](http://sam.pikesley.org/projects/marvel-explorer/) which uses Ultron to [explore](https://twitter.com/marvel_explorer) the [data](http://marvelexplorer.github.io/).

## API

I've tried to follow the [Marvel API](http://developer.marvel.com/docs#!/public/) as closely as possible. It's probably best illustrated with some examples from the [specs](https://github.com/pikesley/ultron/tree/master/spec/ultron):

### Get a thing by its ID

    it 'should find a comic', :vcr do
      comic = Comics.find 12518
      expect(comic.title).to eq ('Uncanny X-Men (1963) #67')
    end

### Get a set of things filtered by some other thing

    it 'should find a list of comics featuring the character', :vcr do
      character = Characters.find 1009610
      expect(character.name).to eq ('Spider-Man')
      comics = Comics.by_character 1009610
      expect(comics.first.title).to eq ('Superior Spider-Man (2013) #22')
      expect(comics.class).to eq (Comics)
      expect(comics.count).to eq (20)
    end

### Get a set of things using query-string parameters

    it 'should let us search with parameters', :vcr do
      comics = Comics.where sharedAppearances: '1009351,1009718' # Hulk and Wolverine
      expect(comics[7].title).to eq ('Deadpool (2008) #37')
    end

### Get a set of things using multiple query-string parameters

    it 'should let us search with multiple parameters', :vcr do
      comics = Comics.where sharedAppearances: '1009610,1009718', events: 302 # Spider-Man and Wolverine, Fear Itself
      expect(comics.first.title).to eq ('Fear Itself (2010) #7')
    end

### Get a set of things filtered by another thing and with a query-string

    it 'should let us get comics by a creator *with params*', :vcr do
      comics = Comics.by_creator_and_with 214, dateRange: '1980-01-01,1989-12-31'
      expect(comics.first.resourceURI).to eq ('http://gateway.marvel.com/v1/public/comics/8268')
    end

### Get a random thing

Something like this works, but it's proving very difficult to test it:

    Characters.sample

### Get a random thing from a search

      it 'should give us a random comic', :vcr do
        set = Comics.by_character 1009610
        set.stub(:random_offset).and_return(512)
        expect(set.sample.title).to eq ('Amazing Spider-Man (1999) #590')
      end

      it 'should give us a random comic for a more complex search', :vcr do
        set = Comics.by_creator_and_with 214, dateRange: '1980-01-01,1989-12-31'
        set.stub(:random_offset).and_return(99)
        expect(set.sample.title).to eq ('Dazzler (1981) #19')
      end

### Pre-baked custom searches

There's a lot of noise in the data, and I found myself applying the same filters again and again. So:

    it 'should give us regular comics filtered by character', :vcr do
      comics = Comics.by_character_and_vanilla_comics 1009685
      expect(comics.first.title).to eq ('AGE OF ULTRON (2013) #1')
    end

This applies the following search params:

    format: 'comic', formatType: 'comic', noVariants: true

which purges all sorts of hardbacks and reprints and so on from the results.

## Exceptions

### Catch and re-raise a 404

    it 'should throw a 404 (wrapped in a Marvel exception) on a 404', :vcr do
      begin
        comic = Comics.find 1000000 # there are not a million comics
      rescue MarvelException => e
        expect(e.code).to eq (404)
        expect(e.status).to eq ("We couldn't find that comic_issue")
      end
    end

### Raise a custom exception when we do something dumb

    it 'should throw a Resource Not Found exception when we search for something nonsensical', :vcr do
      begin
        characters = Characters.by_creator 186 # characters by creator is a nonsense concept in the Marvel API
      rescue UltronException => e
        expect(e.status).to eq ('Resource does not exist. Check http://developer.marvel.com/docs')
      end
    end

This all got a lot more elegant after a chat over a fry-up with [@floppy](http://github.com/floppy) at the always-superb [First-Step Cafe](https://plus.google.com/100027883675109761806/about?gl=uk&hl=en) in Shoreditch.

And here's a [Gource video](https://vimeo.com/86940385) of the project as of 2014-02-17.
