## 1. Learning outcome
You can explain what a cloud platform provider is and can deploy (parts of) your application to a cloud platform. You integrate cloud services (for example: Serverless computing, cloud storage, container management) into your enterprise application, and can explain the added value of these cloud services for your application.

## 2. Algolia 
Algolia is a hosted search engine, offering full-text, numerical, and faceted search, capable of delivering real-time results from the first keystroke. Algolia's powerful API lets you quickly and seamlessly implement search within your websites and mobile applications. Our search API powers billions of queries for thousands of companies every month, delivering relevant results in under 100ms anywhere in the world.

### 2.1 Setup
To make use of Angolia some setup is required. Besides creating an account installing the dependencies the existing data needs to be imported. Using the `php artisan scout:import` command the existing data can be imported into the angolia index.
![Algolia import](https://user-images.githubusercontent.com/46562627/173346236-d1e881ee-92a8-49c9-bd4a-63b28b52c471.PNG)


### 2.2 Implementation
```php
class Song extends Model
{
    use Searchable;
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'id',
        'name',
        'album_id',
        'genre',
        'resourceLocation',
        'releaseDate',
    ];

    protected $table = "song";

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [

    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [

    ];

    public function albums()
    {
        return $this->belongsTo(Album::class);
    }

    public function ratings()
    {
        return $this->hasMany(Rating::class);
    }
}
```

```php
public function searchSong(Request $request)
{
    $validateData = $request->validate([
        'query' => 'required|string',
    ]);
    $collection = Song::search($validateData['query'])->get();
    $response = new ValidResponse($collection);
    return response()->json($response, 200);
}
```

### 2.3 Test
Creating a route makes it possible to access the endpoint and test the functionality:

![Algolia search](https://user-images.githubusercontent.com/46562627/173346568-5e4a1f00-eac8-4fc2-9581-790c2c43d151.PNG)

Through the analytics page of of Angolia I can check that the request came through:

![Algolia search monitor](https://user-images.githubusercontent.com/46562627/173346753-bda1da85-ece7-43a3-a7c8-461c858acb0a.PNG)

### 2.4 Scaling
Being aware of the pricing of a chosen cloud service is really important. Algolia provides 3 types of plans; Free, Standard or Premium. The difference between Standard or Premium are mostly down to personalisation (personalise search results per user using AI), dynamic re-ranking (boost well performing results), and Relevant sorts (AI sorted results without data duplication). Basically most of the AI functionalities.
The exact price is based on the amount of records and requests. The more records and requests are made, the cheaper they become per singular unit. 

![Algolia Pricing](https://user-images.githubusercontent.com/46562627/173388317-052067ff-f1b2-47cc-a382-6d35f9856c57.PNG)

Because pricing is based on records and searches, the ammount of records dictates a large pertion of the total price that wil only go up with more data. Therefore it is important to think about where the weakest point is in your own search algorithm and only use a service like Algolia there. For smaller data sets or situations where slower responses are not an issue, Algolia will become just another billing item without any desired results.
For the proof of concept within this project I chose to stay with the free tier.

## 3. Azure File Storage 
Azure Files is a shared storage service that lets you access files via the Server Message Block (SMB) protocol, and mount file shares on Windows, Linux or Mac machines in the Azure cloud. You can also cache file shares in on-premises Windows Servers using the Azure File Sync agent.

### 3.1 Setup
To make use of an Azure File Storage some setup is required. Through Fontys it is possible to make use of $100 of free Azure credit.
We start of by setting up a [resource group](https://docs.microsoft.com/nl-nl/azure/azure-resource-manager/management/manage-resource-groups-portal). The resource group bundles al lthe functionalities for a project. In this case just the File storage. I created a `Wolkgeluid` resource group containing a `Wolkgeluid12345` file storage.

![image](https://user-images.githubusercontent.com/46562627/174435514-2092d7fd-5de9-425d-b7d1-0db5771c1b79.png)

### 3.2 Implementation
After installing the proper package using the `composer require academe/laravel-azure-file-storage-driver` command, the enviroment variables can be set up and the `azure-file-storage` disk method can be used to write a file to the file storage.

```php
$path = Storage::disk('azure-file-storage')->put("" ,$request->file('song'));
```

```php
'azure-file-storage' => [
            // The driver provided by this package.
            'driver' => 'azure-file-storage',

            // Account credentials.
            'storageAccount' => env('AZURE_FILE_STORAGE_ACCOUNT'),
            'storageAccessKey' => env('AZURE_FILE_STORAGE_ACCESS_KEY'),

            // The file share.
            // This driver supports one file share at a time (you cannot
            // copy or move files between shares natively).
            'fileShareName' => env('AZURE_FILE_STORAGE_SHARE_NAME'),

            // Optional settings
            'disableRecursiveDelete' => false,
            'driverOptions' => [],
            'root' => 'root/directory', // Without leading '/'
        ],
```

### 3.3 Test
Creating a route makes it possible to access the endpoint and test the functionality:

![image](https://user-images.githubusercontent.com/46562627/174436503-371fd0e3-78c2-40db-a383-0ef107796937.png)

Through the monitoring page of of Azure I can check that the request came through:

![image](https://user-images.githubusercontent.com/46562627/174436581-e96310d5-adf0-4606-8881-43765259bce3.png)

### 3.4 Scaling
As is the case for Algolia, pricing and scalability are important factors when making the choice to use a certain cloud service.
The [pricing](https://azure.microsoft.com/en-us/pricing/details/storage/files/) of azure file storage is mostly based on the amount storage space used. However there are some setting that can be adjusted that change the performance of the service and with it the pricing. For development puposes I decided to stick with the Cool data storage type which runs me about $0.015 per used GiB per month. The Cool tier is mostly meant to be used fior online archive storage scenarios. Hot may be more suitible for general purpose file sharing. For deployment I would go with the Premium fule storage which is excellent for high I/O workloads. This storage tier is offered on a high-performance SSD based storage. However this would be at least 10 times more expensive than the cool tier for data at rest.

My current configuration makes use of the "Cool" tier:

![image](https://user-images.githubusercontent.com/46562627/174437092-51e70fa7-6d79-4809-b127-4cc4c96dbd86.png)

![image](https://user-images.githubusercontent.com/46562627/174437200-e349bcc5-a529-4add-9462-0354a5da1cbf.png)


Besides rest data pricing we also have to keep in mind the [pricing](https://azure.microsoft.com/en-us/pricing/details/storage/files/) for the transactions. When going for premium the pricing for the transfers are included and don't have to be paid seperately. On other tiers every 10,000 transactions are charged

![image](https://user-images.githubusercontent.com/46562627/174437228-c4554c58-2692-4455-9183-580602772a65.png)

Lastly you also have to keep in mind that pricing will differ per chosen region. Choosing a cheaper region might come with a result of bigger latency. If a worldwide release is part of the plan it might be worth while to figure out if every region needs it's own storage (which also requires synching at additional cost) or whether the latency is acceptable.
Within this project I chose to stay with the cheapest configuration to make as much use as possible of my $125 through my fontys account
