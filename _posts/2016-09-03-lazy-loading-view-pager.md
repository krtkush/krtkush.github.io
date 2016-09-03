---
layout: post
title: Lazy loading ViewPager tabs
---

In this post I share how I built a ViewPager with dynamic tab addition.

The configuration of a ViewPager in an Android app are often pre-defined; we declare the number of tabs, their respective titles and the fragments they are supposed to load. Such was not the case when I had to implement ViewPager in the Mars Explorer app. I had to make tabs for each [SOL](https://en.wikipedia.org/wiki/Timekeeping_on_Mars). With the current SOL count being 1448 and it only increasing everyday, making so many tabs in one go would be unwise. I concluded lazy loading would be a better approach i.e. add a new tab dynamically when a last few of existing tabs are accessed.

**Requirements:**

1. Load 10 tabs by default.
2. Each tab is supposed to represent an SOL with the latest SOL in the right most tab.
3. When the user reaches the second last or the last tab, add one more tab to the ViewPager.

<br>
We'll also need a fragment class but, the code inside it is not needed for the explanation.

**Setup:**

We'll have the following classes -

1. HostActivity: An activity which will host the ViewPager.
2. ViewPagerAdapter: The adapter which will define the ViewPager behavior.
3. SolFragment: The fragment which will be associated with each tab.

<br>
The code in each of these activities is shown below with appropriate comments.

**HostActivity**

    public void prepareAndImplementViewPager(final ViewPager viewPager,
        final TabLayout tabLayout) {

        String roverName;
        // The maximum available SOL
        String roverSol;
        // Variable to keep track of how many SOLs have had their respective
        // tabs added to the viewPager.
        int roverSolTracker;

        final int numberOfInitialTabs = 10;
        final int numberOfTabsLeftAfterWhichToAdd = 2;

        final List<Fragment> fragmentList = new ArrayList<>();
        final List<String> solList = new ArrayList<>();
        final TabData tabData = new TabData();

        roverSolTracker = Integer.valueOf(roverSol);

        // Initiate and three fragments for
        // the last three SOLs respectively.
        for(int fragmentCount = roverSolTracker;
            fragmentCount > Integer.valueOf(roverSol) - numberOfInitialTabs;
            fragmentCount--) {

        // Arguments to be sent to the fragment
        Bundle args = new Bundle();
        args.putInt(RoverExplorerConstants.roverSolTrackExtra,
          roverSolTracker);
        args.putString(RoverExplorerConstants.roverNameExtra, roverName);

        fragmentList.add(Fragment.instantiate(activity,
                RoverExplorerFragment.class.getName(), args));
        solList.add(String.valueOf(roverSolTracker));
        tabData.setFragmentList(fragmentList);
        tabData.setSolList(solList);

        roverSolTracker--;
    }

    final ViewPagerAdapter viewPagerAdapter =
            new ViewPagerAdapter(activity.getSupportFragmentManager(), tabData);
    viewPager.setAdapter(viewPagerAdapter);
    tabLayout.setupWithViewPager(viewPager);
    viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
        @Override
        public void onPageScrolled(int position, float positionOffset,
                                   int positionOffsetPixels) {

        }

        @Override
        public void onPageSelected(int position) {

            // Check if the user has reached the second last or last tab.
            // If he/ she has and the SOL is not below 0, add another tab.
            if(fragmentList.size() - position <= numberOfTabsLeftAfterWhichToAdd
                    && roverSolTracker >= 0) {

                // Arguments to be sent to the fragment
                Bundle args = new Bundle();
                args.putInt(RoverExplorerConstants.roverSolTrackExtra,
                            roverSolTracker);
                args.putString(RoverExplorerConstants.roverNameExtra, roverName);

                fragmentList.add(Fragment.instantiate(activity,
                        RoverExplorerFragment.class.getName(), args));
                solList.add(String.valueOf(roverSolTracker));
                tabData.setFragmentList(fragmentList);
                tabData.setSolList(solList);

                // Let the adapter know that a new tab has been added
                viewPagerAdapter.notifyDataSetChanged();
                roverSolTracker--;
            }
        }

        @Override
        public void onPageScrollStateChanged(int state) {

        }
    });
  }

**ViewPagerAdapter**

    public class ViewPagerAdapter extends FragmentPagerAdapter {

      private TabData tabData;

      public ViewPagerAdapter(FragmentManager fragmentManager,
                              TabData tabData) {
           super(fragmentManager);
           this.tabData = tabData;
      }

      @Override
      public Fragment getItem(int position) {
          return tabData.getFragmentList().get(position);
      }

      @Override
      public int getCount() {
          return tabData.getFragmentList().size();
      }

      @Override
      public CharSequence getPageTitle(int position) {
          return  "SOL " + tabData.getSolList().get(position);
      }
    }

`TabData` is a custom pojo class which holds a list of fragments and a list of SOL names.
The fragment list is used to inflate the fragments and SOL names are set as tab titles.

With this we have a ViewPager which gets more tabs added to it on demand. Technically, it works pretty much like a RecyclerView or ListView but, I this was the first time I implemented it.
