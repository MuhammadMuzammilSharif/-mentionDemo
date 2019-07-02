# How to create @ Mention (like Linkedin & Facebook) in Android

<!--recently a task is given to me to create a @Mention and #Tag rich Text Editor-->

## Implementation

### Gradle

```
implementation 'com.linkedin.android.spyglass:spyglass:1.4.0'
```
### xml
```
<com.linkedin.android.spyglass.ui.MentionsEditText
     android:id="@+id/rich_text_editor"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:hint="What's on your mind?"
     android:inputType="textMultiLine"
     android:minLines="2" />
```
### Java Initialization 
```
editText = (MentionsEditText) findViewById(R.id.activity_wall_new_post_des_edt);

editText.setTokenizer(new WordTokenizer(new WordTokenizerConfig.Builder().
                setExplicitChars("#@").setThreshold(1).setMaxNumKeywords(1).build()));
        mentionTagAdapter = new MentionAdapter(new ItemClickListener(){
            public void onItemClick(Object object, int pos, int type){
                mentionTagList.setVisibility(View.GONE);
                if (type == 0) {
                    editText.insertMention((Firend) obj);
                } else {
                    editText.insertMention((Tag) obj);
                }
            }
        });
        mentionTagList.setAdapter(mentionTagAdapter);
        mentionTagList.setVisibility(View.GONE);

        editText.setQueryTokenReceiver(new QueryTokenReceiver() {
            @Override
            public List<String> onQueryReceived(@NonNull QueryToken queryToken) {
                if (queryToken.getExplicitChar() == '#') {
                    showHashTagList(queryToken.getKeywords().toLowerCase());
                } else if (queryToken.getExplicitChar() == '@') {
                    showMentionList(queryToken.getKeywords().toLowerCase());
                }
                return null;
            }
        });
```

### ItemClickListener
```
interface ItemClickListener {
    public void onItemClick(Object object, int pos, int type);
}
```
### Show Mentions List (below edit field)
```        
private void showMentionList(String keyword) {
    List<Firend> friends = new ArrayList<>();
    for (Firend friend : ALL_FRIENDS_LIST) {
        if (friend.getName().replaceAll(" ", "").toLowerCase().contains(keyword.toLowerCase())) {
            friends.add(friend);
        }
    }
    if (!friends.isEmpty()) {
        //here show list of tag friends in list bellow editfield
        mentionTagAdapter.setMentions(friends);
        mentionTagList.setVisibility(View.VISIBLE);
        mentionTagAdapter.notifyDataSetChanged();
    } else {
        //here hide mentionUser (ListView/RecyclerView) view
        mentionTagList.setVisibility(View.GONE);
    }
}
```
### Show HashTags List (below edit field)
```
private void showHashTagList(String keyword) {
    List<Tag> tags = new ArrayList<>();
    for (Tag tag : ALL_TAGS_LIST) {
        if (tag.getTitle().replaceAll(" ", "").toLowerCase().contains(keyword.toLowerCase())) {
            tags.add(tag);
        }
    }
    if (!tags.isEmpty()) {
        mentionTagAdapter.setTags(tags);
        mentionTagList.setVisibility(View.VISIBLE);
        mentionTagAdapter.notifyDataSetChanged();
    } else {
        mentionTagList.setVisibility(View.GONE);
    }
}
```

### geting Final html text

this method will generate html with link @Mentions and #tags
```
private String getPostDescription() {
    StringBuilder builder = new StringBuilder();
    MentionsEditable editable = editText.getMentionsText().trim();
    if (editable.getMentionSpans() == null || editable.getMentionSpans().isEmpty()) {
        return "<p>" + editable.toString() + "</p>";
    }
    int start = 0;
    builder.append("<p>")
    for (MentionSpan span : editable.getMentionSpans()) {
        int temp = editable.getSpanStart(span);
        builder.append(editable.toString(), start, temp);
        start = temp;
        if (span.getMention() instanceof Firend) {
            Firend user = (Firend) span.getMention();
            builder.append("<a href=\"/users/" + user.getId() + "\">" + user.getSuggestiblePrimaryText() + "</a>");
        } else {
            Tag user = (Tag) span.getMention();
            builder.append("<a href=\"/search?q=#" + user.getSuggestiblePrimaryText() + "\">#" + user.getSuggestiblePrimaryText() + "</a>");
        }
        temp = editable.getSpanEnd(span);
        if (temp > (editable.toString().length() - 1)) {
            builder.append(editable.toString().substring(start));
        } else {
            builder.append(editable.toString(), start, temp);
        }
        start = temp;
    }
    if (start < editable.toString().length()) {
        builder.append(editable.toString().substring(start));
    }
    builder.append("</p>")
    return builder.toString().replaceAll("\n","<br/>");
}
```

**Note**: **`Friend`** and __`Tag`__ classes must be `implements` __`com.linkedin.android.spyglass.mentions.Mentionable`__ as below

### Friend

```
import android.os.Parcel;
import android.os.Parcelable;
import android.support.annotation.NonNull;

import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;
import com.linkedin.android.spyglass.mentions.Mentionable;

import java.io.Serializable;

public class Friend implements Mentionable, Serializable {

    @SerializedName("id")
    @Expose
    private Integer id;
    @SerializedName("first_name")
    @Expose
    private String firstName;
    @SerializedName("last_name")
    @Expose
    private String lastName;
    @SerializedName("email")
    @Expose
    private String email;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        if (lastName == null || lastName.toLowerCase().trim().equals("null")) {
            return "";
        }
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @NonNull
    @Override
    public String getTextForDisplayMode(MentionDisplayMode mode) {
        switch (mode) {
            case FULL:
                return getFirstName() + " " + getLastName();
            case PARTIAL:
            case NONE:
            default:
                return "";
        }
    }

    @Override
    public MentionDeleteStyle getDeleteStyle() {
        return MentionDeleteStyle.PARTIAL_NAME_DELETE;
    }

    @Override
    public int getSuggestibleId() {
        return getId();
    }

    @Override
    public String getSuggestiblePrimaryText() {
        if (getLastName().trim().length() > 0) {
            return getFirstName() + " " + getLastName();
        }
        return getFirstName();
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(getFirstName());
    }

    public Friend(Parcel in) {
        firstName = in.readString();
    }

    public static final Parcelable.Creator<Friend> CREATOR
            = new Parcelable.Creator<Friend>() {
        public Friend createFromParcel(Parcel in) {
            return new Friend(in);
        }

        public Friend[] newArray(int size) {
            return new Friend[size];
        }
    };
}
```

### Tag

```

import android.os.Parcel;
import android.os.Parcelable;
import android.support.annotation.NonNull;

import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;
import com.linkedin.android.spyglass.mentions.Mentionable;

public class Tag implements Mentionable {
    @SerializedName("title")
    @Expose
    private String title;
    
    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public MentionDeleteStyle getDeleteStyle() {
        return MentionDeleteStyle.PARTIAL_NAME_DELETE;
    }

    @Override
    public int getSuggestibleId() {
        return (int) System.currentTimeMillis();
    }

    @Override
    public String getSuggestiblePrimaryText() {
        return getTitle();
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(getTitle());
    }

    public Tag(Parcel in) {
        title = in.readString();
    }

    public static final Parcelable.Creator<Tag> CREATOR
            = new Parcelable.Creator<Tag>() {
        public Tag createFromParcel(Parcel in) {
            return new Tag(in);
        }

        public Tag[] newArray(int size) {
            return new Tag[size];
        }
    };

    @NonNull
    @Override
    public String getTextForDisplayMode(MentionDisplayMode mode) {
        switch (mode) {
            case FULL:
                return getTitle();
            case PARTIAL:
            case NONE:
            default:
                return "";
        }
    }
}
```
### MentionAdapter
```
import android.util.TypedValue;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import org.jetbrains.annotations.NotNull;
import android.support.v7.widget.RecyclerView;

import java.util.List;

public class MentionAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    public MentionAdapter(ItemClickListener itemClickListener) {
        this.itemClickListener = itemClickListener;
    }

    private ItemClickListener itemClickListener;
    private List<Friend> mentions = null;
    private List<Tag> tags = null;

    private boolean isMentions = true;

    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        TextView tv = new TextView(parent.getContext());
        tv.setLayoutParams(new RecyclerView.LayoutParams(RecyclerView.LayoutParams.MATCH_PARENT,
                RecyclerView.LayoutParams.WRAP_CONTENT));
        tv.setTextSize(TypedValue.COMPLEX_UNIT_SP, 16f);
        tv.setPadding(15, 10, 15, 10);
        return new RecyclerView.ViewHolder(tv) {
        };
    }

    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, final int position) {
        if (isMentions) {
            ((TextView) holder.itemView).setText(mentions.get(position).getSuggestiblePrimaryText());
        } else {
            ((TextView) holder.itemView).setText(tags.get(position).getSuggestiblePrimaryText());
        }
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (isMentions) {
                    itemClickListener.onItemClick(mentions, position, 0);
                } else {
                    itemClickListener.onItemClick(mentions, position, 1);
                }
            }
        });
    }

    public void setMentions(List<Friend> friends) {
        mentions = friends;
        isMentions = true;
        notifyDataSetChanged();
    }

    public void setTags(List<Tag> tags) {
        this.tags = tags;
        isMentions = false;
        notifyDataSetChanged();
    }

    @Override
    public int getItemCount() {
        if (isMentions) {
            if (mentions == null) {
                return 0;
            } else {
                return mentions.size();
            }
        } else {
            if (tags == null) {
                return 0;
            } else {
                return tags.size();
            }
        }
    }
}
```
