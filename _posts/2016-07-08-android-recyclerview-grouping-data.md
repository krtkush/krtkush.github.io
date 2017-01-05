---
layout: post
comments: true
title: Android RecyclerView - Grouping Data
---

This post was featured in [Android Weekly #238 issue](http://androidweekly.net/issues/issue-238).

Recently, I was asked to implement recycler view in an interesting way - group data within a list with respect to a specific attribute of the list elements and also show the grouping attribute tag on top of each group. The closest comparison I can make is how WhatsApp groups together chats with respect to dates. In my case I had to group by months. I found the problem pretty interesting and decided to document my implementation.

<img src="http://i.imgur.com/7cWye6C.png" width="200" height="350" />

In the above image we can see that the chats are divided into groups by dates - "25 June 2016", "2 July 2016" etc.
This is what we wish to achieve.  

**Response from the server**

Here is an example JSON response that is to be populated as a list.

    {
      "list_elements_array": [
        {
          "key_1": "abc",
          "key_2": "xyz",
          "key_3": "Title",
          "key_4": "2016-06-21"
        },
        {...},
        {...},
      ]
    }


The JSON has to be converted into a list using `key_4` as the grouping parameter i.e. all JSON Objects having `2016-06-21` as their respective `key_4` value need to be put under `2016-06-21` group and so on.

**(i) Group data into a HashMap according to the grouping parameter.**
We create a HashMap whose key is the group label and the value is a POJO for the JSON Objects.

Example HashMap -

    HashMap<String, List<PojosOfJsonArray>>

    values -

    "2016-06-21" -> listOfJsonArrayPojo[3]
    "2016-06-05" -> listOfJsonArrayPojo[1]
    "2016-05-17" -> listOfJsonArrayPojo[1]


We can see three groups in the HashMap above - `2016-06-21` with three objects under it, `2016-06-05` with one object under it and, `2016-05-17` also with one object under it.

How do we create such a HashMap?  

We have a method which accepts the server response curated in a list, loops through it and spews out a HashMap.

    private HashMap<String, List<PojoOfJsonArray>>
    groupDataIntoHashMap(List<PojoOfJsonArray> listOfPojosOfJsonArray) {

      HashMap<String, List<PojoOfJsonArray>> groupedHashMap = new HashMap<>();

      for(PojoOfJsonArray pojoOfJsonArray : listOfPojosOfJsonArray) {

          String hashMapKey = listOfPojosOfJsonArray.getKey4Value();

          if(groupedHashMap.containsKey(hashMapKey)) {
              // The key is already in the HashMap; add the pojo object
              // against the existing key.
              groupedHashMap.get(hashMapKey).add(pojosOfJsonArray);
          } else {
              // The key is not there in the HashMap; create a new key-value pair
              List<PojoOfJsonArray> list = new ArrayList<>();
              list.add(listOfPojosOfJsonArray);
              groupedHashMap.put(hashMapKey, list);
          }
      }

      return groupedHashMap;
    }

**(ii) Creating required data structures for the RecyclerView.**
Now we make an abstract class called `ListItem` that will be inherited by two classes which will represent the two different types of list items - `DateItem`, which will show the month and the `GeneralItem` which will show the required data.

  ListItem -

    public abstract class ListItem {

      public static final int TYPE_DATE = 0;
      public static final int TYPE_GENERAL = 1;

      abstract public int getType();
    }

  GeneralItem -

      public class GeneralItem extends ListItem {

        private PojoOfJsonArray pojoOfJsonArray;

        public PojoOfJsonArray getPojoOfJsonArray() {
            return pojoOfJsonArray;
        }

        public void setPojoOfJsonArray(PojoOfJsonArray pojoOfJsonArray) {
            this.pojoOfJsonArray = pojoOfJsonArray;
        }

        @Override
        public int getType() {
          return TYPE_GENERAL;
        }
      }

  DateItem -

    public class DateItem extends ListItem {

      private String date;

      public String getDate() {
        return date;
      }

      public void setDate(String date) {
        this.date = date;
      }

      @Override
      public int getType() {
        return TYPE_DATE;
      }
    }


**(iii) Converting the HashMap data into a sorted list.**
Next, we consolidate the HashMap into a ArrayList (of our newly created `ListItem` type) called `consolidatedList` with `DateItem` injected between `GeneralItem` at the required places.  

    // We linearly add every
    List<ListItem> consolidatedList = new ArrayList<>();

    for (String date : groupedHashMap.keySet()) {
      DateItem dateItem = new DateItem();
      dateItem.setDate(date);
      consolidatedList.add(dateItem);

      for (PojoOfJsonArray pojoOfJsonArray : groupedHashMap.get(date)) {
        GeneralItem generalItem = new GeneralItem();
        generalItem.setBookingDataTabs(bookingDataTabs);
        consolidatedList.add(generalItem);
      }
    }

We pass `consolidatedList` to the constructor of the RecyclerView adapter.

**(iv) Defining the RecyclerView.**
We now define our `RecyclerView` which will check which type layout is to be populated depending on type of `ListItem` (TYPE_DATE or TYPE_GENERAL); We'll need to create two ViewHolders for the respective layouts.

    public class GroupedListRecyclerAdapter
      extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

          private List<ListItem> consolidatedList = new ArrayList<>();

          public GroupedListRecyclerAdapter(List<ListItem> consolidatedList) {
            this.consolidatedList = consolidatedList;
          }

          @Override
          public int getItemViewType(int position) {
            return consolidatedList.get(position).getType();
          }

          @Override
          public int getItemCount() {
            return consolidatedList != null ? consolidatedList.size() : 0;
          }

          @Override
          public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent,
                                                            int viewType) {

            RecyclerView.ViewHolder viewHolder = null;
            LayoutInflater inflater = LayoutInflater.from(parent.getContext());

            switch (viewType) {

              case ListItem.TYPE_BOOKING:
                  View v1 = inflater.inflate(R.layout.general_layout, parent,
                                              false);
                  viewHolder = new GeneralViewHolder(v1);
                  break;

              case ListItem.TYPE_DATE:
                  View v2 = inflater.inflate(R.layout.date_layout, parent, false);
                  viewHolder = new DateViewHolder(v2);
                  break;
            }

            return viewHolder;
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder viewHolder,
                                      int position) {

                switch (viewHolder.getItemViewType()) {

                    case ListItem.TYPE_BOOKING:
                        GeneralItem generalItem
                        = (GeneralItem) consolidatedList.get(position);
                        GeneralViewHolder generalViewHolder
                        = (GeneralViewHolder) viewHolder;

                        // Populate general item data here

                        break;

                    case ListItem.TYPE_DATE:
                        DateItem dateItem
                        = (DateItem) consolidatedList.get(position);
                        DateViewHolder dateViewHolder
                        = (DateViewHolder) viewHolder;

                        // Populate date item data here

                        break;
                }
            }
        }

        // ViewHolder for date row item
        class DateViewHolder extends RecyclerView.ViewHolder {
            ...
            public DateViewHolder(View v) {
                super(v);
                ...
            }
        }

        // View holder for general row item
        class GeneralViewHolder extends RecyclerView.ViewHolder {
            ...
            public GeneralViewHolder(View v) {
                super(v);
                ...
            }
        }
    }

You'll notice that we are overriding `getItemViewType`, a method we usually don't override. This method is responsible for informing the adapter which type of row to work on.

The other thing to notice is how both `onCreateViewHolder` and `onBindViewHolder` have a switch case. These are there for same reasons as above - informing the adapter which type of row to work on.
