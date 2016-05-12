---
layout: post
title: Delving into Butterknife
---

Butterknife comes up right on top of my 'Basic Android libraries to use' list whenever I'm setting up a new Android project. It is fairy simple to use and straight forward to implement. It helps me get rid of lot of biolerplate code and makes my code look pretty and easy to understand. I highly recommend it.

I wanted to share some not so straight forward stuff I learnt about Butterknife while working with it.

**Using OnItemSelected listener in case of multiple spinners in the same activity.**

Butterknife provides a very nifty way to apply a common listener to multiple views by bundling them together. I found it a bit tricky to leverage this feature in case of multiple spinners in the same activity. The code is pretty straight forward but it took me time to realize that `((View) view.getParent()).getId()` does the job of getting ID of the spinner that has been selected and not `view.getId()`.

    ```Java
    @OnItemSelected({R.id.spinner1, R.id.spinner2})
    public void cancellationPolicySelected(int position, View view) {

        switch (((View) view.getParent()).getId()) {

            case R.id.spinner1:
                Log.i("Position selected",
                String.valueOf(position));
            break;

            case R.id.spinner2:
                Log.i("Position selected",
                String.valueOf(position));
            break;
        }
    }
    ```



**Using Butterknife with Fragments and \<include\> tag**

Things get a little interesting when you want to access a fragment's parent activity view from the fragment itself.

`getActivity().findViewById(R.id.viewId) ...` usually does the job when not using butterknife. Using `@Bind(R.id.viewId) ...` on the fragment will result in a Nullpointer Exception and your app crashing. The way to use butterknife in such a situation is like this -

    ```Java
    public class FragmentClass extends Fragment {

    /**
    * This class is needed to access the views of
    * the host activity (Specifically for butterknife)
    */
    public class HostActivityViews {

        @Bind(R.id.nextButton) Button nextButton;

        @OnClick(R.id.nextButton)
        public void next() {
            Log.i("Button tapped", "Next");
        }

        /**
        * Instance of class that holds views of host activity
        */
        private HostActivityViews hostActivityViews;

        @Override
        public View onCreateView(LayoutInflater inflater,
                ViewGroup container, Bundle savedInstanceState) {

            ...

            hostActivityViews = new HostActivityViews();
            //Bind the views of the fragment
            ButterKnife.bind(this, view);
            // Bind the views of the parent activity
            ButterKnife.bind(hostActivityViews, getActivity());

            ...
        }

        ...

        @Override
        public void onDestroyView() {
            super.onDestroyView();
            ButterKnife.unbind(this);
            ButterKnife.unbind(hostActivityViews);
        }
    }
    ```


A similar approach can be applied for cases in which the XML layout uses the \<include\> tag.

XML layout
    ```XML
    <include
        android:id="@+id/includedLayout"
        android:layout_alignParentTop="true"
        android:layout_height="wrap_content"
        android:layout_width="match_parent"
        layout="@layout/common_layout" />
    ```

Java code -

    ```Java
    public class MainActivity extends AppCompatActivity {

        @Bind(R.id.includedLayout) View includedLayout;

        /**
        *  Class to access the toolbar in the
        * included layout (Specifically for butterknife)
        */
        public class CommonLayout {
        // Here toolbar is the view that is inside common_layout
        @Bind(R.id.toolbar) Toolbar toolbar;
        }

        /**
        * Instance of class that holds views of the included layout
        */
        CommonLayout commonLayout;

        @Override
        public View onCreate(Bundle savedInstanceState) {
        ...

        commonLayout = new CommonLayout();

        // Bind the views of the activity
        ButterKnife.bind(this);
        // Bind the views of the layout that has been included
        ButterKnife.bind(commonLayout, includedLayout);

        ...
        }

    ...
    }
    ```

I'm not sure how useful this whole approach is because writing so much code reduces the major advantage of Butterknife - the ability to get rid of boilerplate code. `getActivity().findViewById(R.id.viewId) ...` seems to be a more straight forward approach.
