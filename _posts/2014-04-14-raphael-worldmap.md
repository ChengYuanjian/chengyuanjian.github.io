---
layout:         post
title:         使用Raphael4GWT画世界地图
description: Raphael JS库通过SVG/VML+JS实现跨浏览器的矢量图形实现方案，而Raphael4GWT是Raphael JS的基于GWT的实现方案。本文以世界地图为例进行演示。
keywords: GWT,Java,Raphael
category: GWT
tags: [GWT]
---

Raphael JS库通过SVG/VML+JS实现跨浏览器的矢量图形实现方案，而Raphael4GWT是Raphael JS的基于GWT的实现方案。本文以世界地图为例进行演示。

###下载Raphael4GWT包

[点击查看](http://code.google.com/p/raphael4gwt/downloads/list)下载列表，本例使用raphael4gwt-0.40.jar。

<!-- more -->

###构造world.js

保存了每个国家的svg路径和名称对应，[点击查看](https://github.com/ChengYuanjian/chengyuanjian.github.io/blob/master/js/world.js)。

###引入Raphael4GWT

`<inherits name="org.sgx.raphael4gwt.GRaphael4Gwt" />`

`<script type="text/javascript" src="js/world.js"></script>`

###使用JSNI构造解析类WorldMapData.java

{% highlight java %}
import com.google.gwt.core.client.JavaScriptObject;

public class WorldMapData {
	private static String[] countryIds = null;
	
	public static native String getCountryShape(String countryId)/*-{
		return $wnd.worldmap.shapes[countryId];
	}-*/;
	
	public static String[] getCountryIds(){
		if(countryIds == null){
			JavaScriptObject jsIds = getCountryIdsJs();
			countryIds = convertToJavaStringArray(jsIds);
		}
		return countryIds;
	};

	public static native String getCountryName(String countryId)/*-{
		return $wnd.worldmap.names[countryId];
	}-*/;
	
	private static native int getArrayLength(JavaScriptObject array) /*-{
	    return array.length;
	}-*/;
	
	private static native String getStringArrayValue(JavaScriptObject obj, int idx) /*-{
	    return obj[idx];
	}-*/;
	
	private static native JavaScriptObject getCountryIdsJs() /*-{
	    var countries = [];
	    var i=0;
	    for(var field in $wnd.worldmap.names){
	    	countries[i++] = field;
	    }
	    return countries;
	}-*/;
	
	private static String[] convertToJavaStringArray(JavaScriptObject array) {
        int length = getArrayLength(array);
        String[] arr = new String[length];
        for (int i = 0; i < length; i++) {
            arr[i] = getStringArrayValue(array, i);
        }
        return arr;
    }
}
{% endhighlight %}

###具体实现

*构造底层容器*

{% highlight java %}LayoutPanel	container	= new LayoutPanel();{% endhighlight %}

*基于容器构造画布*

{% highlight java %}Paper paper = Raphael.paper(container);{% endhighlight %}

*构造HashMap保存所有国家*

{% highlight java %}HashMap<String, Shape>	countryShapes	= new HashMap<String, Shape>();{% endhighlight %}

*遍历国家路径数据，开始作图*

{% highlight java %}
Shape drawCountry(Paper paper, String countryId) {
		String shapeData = WorldMapData.getCountryShape(countryId);
		return paper.path(shapeData).setAttribute("stroke", COUNTRY_FILL).setAttribute("fill", COUNTRY_FILL)
				.setAttribute("opacity", 1);
	}
	
for (String countryId : WorldMapData.getCountryIds()) {
			countryShapes.put(countryId, drawCountry(paper, countryId));
		}
	
{% endhighlight %}

###添加鼠标事件

*右键单击*

{% highlight java %}
MouseEventListener createRightClickListener() {
		return new MouseEventListener() {

			@Override
			public void notifyMouseEvent(NativeEvent e) {
				int k = (e.getButton() & NativeEvent.BUTTON_RIGHT);
				if (k > 0)
					//TODO
			}
		};

	}
	
	countryShapes.get("CN").mouseDown(createRightClickListener());//以中国为例
{% endhighlight %}

*鼠标悬停*

{% highlight java %}
//以中国为例
countryShapes.get("CN").hover(new HoverListener() {
			@Override
			public void hoverOut(NativeEvent e) {
				
			}

			@Override
			public void hoverIn(NativeEvent e) {
							
			}
		});
{% endhighlight %}

效果图，可参加[官方实例](http://raphaeljs.com/world/)。
