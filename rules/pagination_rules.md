# Pagination Rules (Project Standard)

This project uses a custom pagination system controlled from the app side.
Do NOT rely on API pagination response such as totalPages, totalItems, etc.
Pagination is determined by the number of items returned from the API.

---

# 1. Pagination Concept

Pagination will work based on item count:

* If returned items == limit → next page exists
* If returned items < limit → no more pages
* Page number will be increased manually in Bloc
* API pagination response will be ignored

Example:

* limit = 10
* API returns 10 items → load next page
* API returns 10 items → load next page
* API returns 5 items → stop pagination

---

# 2. PaginationParams (Shared)

Pagination must always be sent using PaginationParams.

PaginationParams contains:

* page
* limit

PaginationParams must be passed from:
UI → Bloc → UseCase → Repository → RemoteDataSource → API

Example structure:

```
PaginationParams {
  int page;
  int limit;
}
```

---

# 3. Pagination With Other Query Parameters

If API requires filters/search/sorting along with pagination:

Create FeatureParams and include PaginationParams inside it.

Example:

```
FeatureParams {
  PaginationParams pagination;
  String? search;
  String? gender;
  String? categoryId;
}
```

Query must be merged like:

```
{
  ...pagination.toQuery(),
  'search': search,
  'gender': gender,
  'categoryId': categoryId
}
```

Final API Query Example:

```
?page=1&limit=10&search=shirt&gender=male
```

---

# 4. Bloc Pagination Variables

Every paginated Bloc must contain:

```
int page = 1;
int limit = 10;
bool hasMore = true;
bool isLoadingMore = false;
List<Entity> items = [];
```

---

# 5. Bloc Pagination Flow

## First Load

1. page = 1
2. Call API
3. Replace items list
4. If returnedItems < limit → hasMore = false
5. Else → page++

## Load More

1. If hasMore == false → stop
2. If isLoadingMore == true → stop
3. Call API with next page
4. Append new items to existing list
5. If returnedItems < limit → hasMore = false
6. Else → page++

---

# 6. Repository Rules

Repository must:

* Accept Params (with PaginationParams)
* Call API with queryParameters
* Return List<Entity>
* NOT handle pagination logic
* NOT increment page
* NOT decide hasMore

Pagination logic must only exist in Bloc.

Example:

```
Future<Either<Failure, List<Entity>>> getItems(FeatureParams params);
```

---

# 7. Remote Data Source Rules

RemoteDataSource must:

* Send page and limit in query
* Parse response data list
* Return List<Model>
* Ignore pagination object from API response

Do NOT use:

* totalPages
* totalItems
* currentPage from API

---

# 8. Load More Rules

When loading more data:

* Do NOT replace existing list
* Append new items
* Maintain old items
* Update page only when items == limit
* Stop when items < limit

Correct:

```
items.addAll(newItems);
```

Wrong:

```
items = newItems;
```

---

# 9. Stop Pagination Conditions

Pagination must stop when:

* returnedItems < limit
* returnedItems == 0
* hasMore == false

---

# 10. Standard Pagination Summary

Pagination Standard Flow:

1. Send page and limit in API request
2. Receive list of items
3. If list length == limit → next page available
4. Increase page
5. Load more when user scrolls
6. Append new items
7. If list length < limit → stop pagination
8. Ignore API pagination response
9. Pagination logic must be in Bloc only

---

# 11. Mandatory Project Pagination Rules

These rules must be followed in all features:

* Always use PaginationParams
* Always send page and limit
* Always control pagination from Bloc
* Never rely on API pagination response
* Always append items on load more
* Never duplicate pagination logic
* Never implement pagination inside repository
* Never implement pagination inside remote data source
* Pagination must be identical across the entire project

This is the official pagination system for this project.
