---
layout: post
title:  "My Portfolio"
date:   2018-8-28 08:00:00
categories: [portfolio]
---

Bink Wang \|  Senior iOS Developer \| fxmmc@sina.com

LinkedIn profile: https://www.linkedin.com/in/bink-wang-653836a8/

---

### Project 1: Shanghai Disney Resort Mobile App (iOS)

`Description`

This is Shanghai Disney Resort official mobile application.

The main functions including,
* Finder: show a map of the park with of all facilities/attractions.
* Profile: guest can register/login to discover more things.
* Ticketing: purchasing tickets online.
* FastPass: getting free fast pass as well as purchasing premise access.

For more information, please find in App Store: https://itunes.apple.com/nz/app/Shanghai%20Disney%20Resort/id1073826118?mt=8


`Finder UI`

<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/finder-1.png" width="50%" height="50%" />...<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/finder-2.png" width="50%" height="50%" />
<!--![](../pic/portfolio/finder-1.png)-->
<!--![](../pic/portfolio/finder-2.png)-->


`Finder code snippet`

{% highlight ruby %}
@implementation WDPRFinderUI

+ (NSBundle *)finderUIResourceBundle
{
static NSBundle *_finderUIResourceBundle;
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
_finderUIResourceBundle = [NSBundle bundleFromMainBundleOrFramework:WDPRFinderUIResourceFrameworkName
bundleName:WDPRFinderUIResourceBundleName];
});

return _finderUIResourceBundle;
}

+ (NSString *)localizedStringForKey:(NSString *)key
{
return [self.finderUIResourceBundle localizedStringForKey:key value:@"" table: nil];
}

@end
{% endhighlight %}

<!--<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/finder-code-1.png" width="50%" height="50%" />-->
<!--![](../pic/portfolio/finder-code-1.png)-->


`Ticketing UI`

<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/ticketing-1.png" width="50%" height="50%" />...<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/ticketing-2.png" width="50%" height="50%" />
<!--![](../pic/portfolio/ticketing-1.png)-->
<!--![](../pic/portfolio/ticketing-2.png)-->

`Ticketing code snippet`

{% highlight ruby %}
typedef void (^ServiceResponse)(NSError *error);

@interface WDPRPrivateDataService (Booking)

// Create an Order object
/// @param orderParameters is the order parameters
/// @param success is a block called when the service returns with serialized raw roduct entries
/// @param failure is a block called when the service returns with unexpected data
- (void)createOrderWithParameters:(MdxTicketViewModel *)orderParameters
success:(ServiceSuccess)success
failure:(ServiceFailure)failure;

// Create an Order object (same as above but returns a model object)
/// @param orderParameters is the order parameters
/// @param completion either the pending order object or an error will be returned
- (void)createOrderWithParameters:(MdxTicketViewModel *)orderParameters
completion:(void (^) (MdxTicketPendingOrder *pendingOrder, NSError *error))completion;

// Freeze tickets
/// @param orderParameters is the order parameters
/// @param success is a block called when the service returns with serialized raw roduct entries
/// @param failure is a block called when the service returns with unexpected data
- (void)freezeTicketWithParameters:(MdxTicketViewModel *)orderParameters
success:(ServiceSuccess)success
failure:(ServiceFailure)failure;
{% endhighlight %}

<!--<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/ticketing-code-1.png" width="50%" height="50%" />-->
<!--![](../pic/portfolio/ticketing-code-1.png)-->

`Profile UI`

<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/profile-1.png" width="50%" height="50%" />...<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/profile-2.png" width="50%" height="50%" />
<!--![](../pic/portfolio/profile-1.png)-->
<!--![](../pic/portfolio/profile-2.png)-->

`Profile code snippet`

{% highlight ruby %}
@interface WDPREditAddressViewController : WDPRTableController

/**
* Method to get all address fields.
*
* @param address WDPRAddress object
* @param table Instance of WDPRTableController to be used in each field
*
* @return address fields
*/
+ (NSArray *)addressFields:(WDPRAddress *)address
forTable:(WDPRTableController *)table;

/**
* Method to get all address fields setting the delegate to the table.
*
* @param address WDPRAddress object
* @param table Instance of WDPRTableController to be used in each field
*
* @return address fields
*/
+ (NSArray *)addressFieldsWithDelegate:(WDPRAddress *)address
forTable:(WDPRTableController *)table;
{% endhighlight %}

<!--<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/profile-code-1.png" width="50%" height="50%" />-->
<!--![](../pic/portfolio/profile-code-1.png)-->

---

### Project 2: Trade Me Demo (iOS)


`Description`

This is a private demo project, to fetch Trade Me categories and show listings of selected category.
Open source. See detail in my GitHub: https://github.com/hanmbink/TME-Demo

`Architecture`

<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/tem-architecture-1.png" width="50%" height="50%" />
<!--![](../pic/portfolio/tem-architecture-1.png)-->

`UI`

<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/tme-ui-1.png" width="50%" height="50%" />
<!--![](../pic/portfolio/tme-ui-1.png)-->

`Code snippet`

{% highlight ruby %}
func fetchListing(_ catetoryId: String?, completionHandler: @escaping DataCompletionHandler) {
guard let catetoryId = catetoryId, !(catetoryId.isEmpty) else { return }

//--- example: https://api.trademe.co.nz/v1/Search/General.json?category=3720
let url = "\(TMEEndpointGeneralSearch)?category=\(catetoryId)&rows=\(TMEListingPageSize)"
let request = NSMutableURLRequest(url: NSURL(string: url)! as URL,
cachePolicy: .useProtocolCachePolicy,
timeoutInterval: TMERequestTimeoutInterval)
request.httpMethod = "GET"
request.allHTTPHeaderFields = headersWithAuthorization as [String: String]

let dataTask = URLSession.shared.dataTask(with: request as URLRequest, completionHandler: { (data, response, error) -> Void in
completionHandler(data, error)
})
dataTask.resume()
}
{% endhighlight %}

{% highlight ruby %}
/**
Fetch & parse the listings of a specific leaf category.

Return a listing array or an error message to client.
*/
func fetchListing(_ catetoryId: String?, completion: @escaping ListingParsingCompletionHandler) {
TMEDataRequester.shared.fetchListing(catetoryId) { (data, error) -> Void in
TMEDataParser.shared.parseListingSearchResponse(data, error, completion: { (listings, errString) in
completion(listings, errString)
})
}
}
{% endhighlight %}

<!--<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/tme-code-1.png" width="50%" height="50%" />...<img src="https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/portfolio/tme-code-2.png" width="50%" height="50%" />-->
<!--![](../pic/portfolio/tme-code-1.png)-->
<!--![](../pic/portfolio/tme-code-2.png)-->


---

### Project 3: YAWA-Demo

`Description`

This is a demo project in Swift which show weather infomation of specific cities. 
Open source. See detail in my GitHub: https://github.com/hanmbink/YAWA-Demo

---
