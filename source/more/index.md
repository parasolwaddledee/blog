---
layout: page
menu_id: more
breadcrumb: false
comments: false
leftbar: subscriptions
---

{% navbar active:/more/ [我的订阅](/more/) [我的歌单](/more/music/) [关于本站](/more/about/) [关于博主](/more/me/) %}

{% friends subscriptions %}

{% grid %}
<!-- cell -->
{% timeline type:fcircle api:https://subscriptions.waddledee.com/all?rule=created&start=00&end=12 %}
{% endtimeline %}
<!-- cell -->
{% timeline type:fcircle api:https://subscriptions.waddledee.com/all?rule=created&start=12&end=24 %}
{% endtimeline %}
{% endgrid %}

<style>
  .md-text .tag-plugin.timeline .timenode>.body, .md-text .tag-plugin.timeline .timenode>.header {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
</style>
