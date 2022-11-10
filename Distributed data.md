## 1. Learning outcome
You are aware of specific data requirements for enterprise systems. You apply best practices for distributed data during your whole development process, both for non-functional and functional requirements. You especially take legal and ethical issues into consideration.

## 2. GDPR 
It is important to be really careful with personal data. The first step to this is to prevent saving personal data that isn't neccesary in the first place. 

## 3. CAP Theorem
The CAP theorem (Brewer's theorem) states that distributed data can only garuentee 2 of 3 elements with a single system.

- **Consistency:** Every client has the exact same (version of) data or an error
- **Availability:** Every request is always responded to, even if the data might not be as recent as could be
- **Partition tolerence:** A system continues to function even if messages are lost or delayed


here is a visual representation of what these choises look like:

![image](https://user-images.githubusercontent.com/46562627/199088993-d50c453b-7591-423c-8402-e4176a9563e1.png)

NoSQL (non-relational) databases are perfect for distributed data applications. It contrast to it's vertical scalable relational sibling, NoSQL databases can easily be horizontally scaled and become distributed by design.
A database can be defines based on the two CAP garuentees it provides:

- **CP:** Provides consitency and partition tolerance at the cost of availibility. When a partition is created between two nodes, the non-consistant node has te be shut down or made unavailable until the issue has been resolved.
- **AP:** Provides availibility and partition tolerance at the cost of consistency. This means that when a partition occurs and a request is made, data is always returned. But this data might not always be the correct (newest) data.
- **CA:** Provides consistency and availibility at the cost of partition tolerance. This means that a request is always responded, the data is always the most recent data, except for when a partition is created.

To be able to choose a fitting database solution a choise has to be made with consideration to the applications needs.

Since consistancy isn't the main focus for this application, and partitioning is needed to be able to create a distributed system, an AP database is needed. 

Cassandra is a perfect example of an AP database since it is highly available which is important for a user focussed application whe consistancy isnt the main goal. Cassandra consists of mulple nots in a system and is peer-to-peer based. In every node multiple replica's exit. Because of this design it means that there is no single master note which would mean a single point of failure. The replication factor in a database determines into how many nodes the data is replicated as is visible in this picture.

![image](https://user-images.githubusercontent.com/46562627/201076791-3991a3e7-5ba8-498e-afd6-24d3ae580b29.png)

In the case that the nodes do not update correctly and the data is not consistent, data will still be returned to the user. This means that the user could get a music file with for example outdated metadata or an artist bio which hasn't been updated yet. For a user this is rarely a problem if the main goal is just listening or uploading music. The only case in this the missing consitancy could be a problem is when a resourcelocation isn't updated and the user recieves an old resourcelocation which snt available anymore. However when a resource is changed the main goal would be to make the old resource unavailable anyway so the main purpose has been fufilled. Due to eventual consistency all nodes should eventually have the same data. 


## 4. Event Sourcing
Within music streaming services it is really common that songs or even entire albums have to be changed up multiple times. Sometimes to release diffrent versions but sometimes its as simple as having to change a title or some metadata. Having people do this makes it really hard to audit it something goes wrong, it's really hard to trace back the data. Therefore [event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) is a good way to create an audit log. It logs everything that happens to a certain record or entry. The result of retrieving a certain record is therefore never based on a single field or record but a "sum" of all the transactions.

### 4.1 Setup
The API needs an event model and a database migration that matches quite closely with the record to be sourced. In the example provided I have applied the theorie to a song entity. 

### 4.2 Implementation
Firstly an event model is created which is almost a carbon copy to the original. The only thing that is really added is an `action type`. This variable declares what action exactly was undertaken to create this result. 
```php
class SongEvent extends Model
{
    protected $fillable = [
        'id',
        'song_id',
        'action_type',
        'name',
        'album_id',
        'genre',
        'resourceLocation',
        'releaseDate',
    ];

    protected $table = "song_events";
}
```
The [migration](https://laravel.com/docs/9.x/migrations) is merely a way to set up the needed database tables.

![image](https://user-images.githubusercontent.com/46562627/174597793-78902a53-e088-4f5a-8211-b7b6eaa4812b.png)

The uploadSong method has some extra logic now. After uploading the song to the file storage, and creating a song record, a SongEvent record is created. This logs the action of uploading. The same is then applied to anything that can happen to the song from this point out. For example updating, deleting, etc.
```php
$path = Storage::disk('azure-file-storage')->put("" ,$request->file('song'));

$song = new Song([
    'name' => $validateData['title'],
    'genre' => $validateData['genre'],
    'album_id' => $validateData['album_id'],
    'resourceLocation' => $path,
    'releaseDate' => now(),
]);

$album->songs()->save($song);
$songEvent = new SongEvent([
    'song_id' => $song->id,
    'action_type' => 'upload',
    'name' => $song->name,
    'album_id' => $song->album_id,
    'genre' => $song->genre,
    'resourceLocation' => $song->resourceLocation,
    'releaseDate' => $song->releaseDate,
]);
$songEvent->save();
```

### 4.3 Test
We can simply test this out by uploading a song. After the song has been uploaded and upload SongEvent record should have been created:

![image](https://user-images.githubusercontent.com/46562627/199091969-8e861648-e621-48dd-9487-71a0720fd236.png)

And after deleting the song, another event should've been added:

![image](https://user-images.githubusercontent.com/46562627/199091908-ee9ebd63-602b-4778-aeef-1ca115a46053.png)

![image](https://user-images.githubusercontent.com/46562627/199087441-57e8da60-49c0-4df2-9b4a-fb506e74ab9a.png)
