# Rate With Science

A web app for finding a rating scale between two arbitrary things.
Ever wanted to rate something on a scale of David Hasselhoff to the Eiffel Tower? Well now you can.

[Dave Pagurek](http://davepagurek.com/) and I made this for the [TerribleHack Winter 2015](http://terriblehack.website/) hackathon. I wrote the backend, Dave wrote the frontend.

Some time I will put up a public version but for now here's a screenshot:
![Screenshot](http://imgur.com/8df1Ujy.png)

## How It Works

Rate With Science uses shortest path graph searching on the Wikipedia link graph to find connections between different things. It uses an efficient binary link graph format created by my [wikicrush](https://github.com/trishume/wikicrush) project so that I can fit the entire graph into around 500mb of memory and do very fast breadth first searches on it.

It also uses the feature of wikicrush of annotating bidirectional paths. When an article links to another article and that article links back to it, that is indicative of a stronger relationship than a single-directional link. The app first tries to find a path using only bidirectional links and falls back on unidirectional links if it can't. The bidirectional paths generally work out nicer since they don't include tenuous connections like "Wikipedia is headquartered in the San Francisco Bay Area".

Dave's JS sends my backend two endpoints via AJAX and my backend gives back a list of articles forming a path or an error in the rare case it can't find a path.

## The Tech

The frontend is written in static HTML and JS with CSS3 animations and properties for extra fancy design.
The backend uses a compiled Rust binary from the [ratews_backend](https://github.com/trishume/ratews_backend) project to find a path.

Unfortunately the link between them is a total hack brought about by the fact that Rust is alpha quality, I don't really know how to use it, and it was only a one day hackathon. After hitting the wall that [Iron](http://ironframework.io/) couldn't gracefully handle the global graph data and Sqlite connection between requests because the Sqlite connection type wasn't thread-safe and Iron is multithreaded. I could have worked around this by proper multi-threading and message passing but I didn't know how to do that in Rust and time was running out.

The solution was to make the Rust binary interact over stdin and stdout via a simple text format. Then I wrote a simple [Sinatra](http://www.sinatrarb.com/) app in Ruby that served Dave's frontend and shuttled API requests to the rust process over IO pipes and back to the frontend via JSON.

Hackathon's aren't where you go to find clean code...

Also all the paths are hard coded absolute paths to the various component projects on my laptops file system...