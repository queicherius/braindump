# Music recognition services for the browser

Our goal is to

- automatically identify music from the user 
- without having to upload the whole music file
- without having to maintain a server (either for the language or for a database) ourselves
- for free and preferablly open-source

---

#### [audiotag](http://audiotag.info/) and similar

These services identify music based on a file and not a fingerprint, and don't offer any API or data.

---

#### [discogs](http://www.discogs.com/), [spotify](https://developer.spotify.com/) and similar

These services only offer the data attached to music, but no music recognition service.

---

#### [freedb](http://www.freedb.org/), [freedb2](http://freedb2.org/), [gnudb](http://www.gnudb.org/) and similar

These services offer music detection, but only based on a unique compact disc ID and **not** on fingerprints of single files. They also require to download their database and maintain it, and don't offer any APIs.

---

#### [Echoprint](http://echoprint.me/)

This service offers music recognition based on a fingerprint, but requires to set up a server and manage the data ourselves (C++).

---

#### [Gracenote](https://developer.gracenote.com)

This service theoretically satisfies our requirements, but requires the "Gracenote Thin Client" for creating fingerprints, which seems to not be available anywhere. The documentation of this service is terrible, as well.

---

#### [AcoustID](https://acoustid.org)

This service uses the database from [MusicBrainz](http://musicbrainz.org/) (which does **not** have a API for that itself, but would require us maintaining the database). It offers a [free service to get meta data for fingerprints of music files](https://acoustid.org/webservice#lookup). The problem is that the [fingerprinting library](https://acoustid.org/chromaprint) for this tool requires either Python or C.

---

#### [ACRCloud](acrcloud.com)

This service offers [music recognition](https://docs.acrcloud.com/webapi) in different plans - a "free trial" plan for 14 days, where we could send 10.000 recognition requests per day, and a "free" plan, which only has 100 per day. It [is available for javascript](https://github.com/acrcloud/webapi_example/blob/master/identify%20protocol%201%20(recommended)/IdentifyProtocolV1.js) as well. It seems like we have to upload whole files with this or write our own fingerprinter, because the fingerprinter is only available in Java, Python and C#.
