---
layout: page
title: "UILabel + UIScrollView + Auto Layout"
date: 2014-09-13 18:49
comments: true
sharing: true
footer: true
---

## Introduction
In this article I'll explain how to easily fit a `UILabel` of variable height into a `UIScrollView` using Auto Layout, for those scenarios where a `UITextView` won't do.

Why? Because Auto Layout will make sure the label's `frame` and the scrollview's `contentSize` will be the ones you want, regardless of orientation and device size. No more calls to `sizeToFit` or `setContentSize`!

{% img https://i.imgur.com/Dogopki.png 200 %}

## Code

You can go ahead and copy and paste this in an empty view controller to see in action:

```objective-c

- (void)viewDidLoad
{
    [super viewDidLoad];

    /*** Init the scrollview and the label ***/

    UIScrollView *scrollView= [UIScrollView new];
    scrollView.translatesAutoresizingMaskIntoConstraints = NO;
    [self.view addSubview:scrollView];

    UILabel *scrollViewLabel = [[UILabel alloc] init];
    scrollViewLabel.numberOfLines = 0;
    scrollViewLabel.translatesAutoresizingMaskIntoConstraints = NO;
    [scrollView addSubview:scrollViewLabel];

    scrollViewLabel.text = @"Bacon ipsum dolor sit amet drumstick meatloaf filet mignon ham t-bone andouille meatball venison cow capicola jerky shankle shoulder ground round. Shank filet mignon pork chop ham hock, short ribs jerky prosciutto tongue porchetta. Biltong kevin strip steak tail jowl jerky boudin drumstick pastrami bresaola. Sirloin tail shoulder salami, hamburger beef doner turducken chuck boudin kielbasa sausage pork loin. Ball tip leberkas fatback, pork chop tail ham ribeye. Bresaola pancetta jerky beef kielbasa frankfurter, corned beef filet mignon ribeye tongue porchetta. Prosciutto short loin sirloin doner brisket jerky swine sausage bresaola chuck. Meatloaf pork chop ribeye bacon jerky turducken, andouille pork belly beef ribs ham hock leberkas. Andouille tri-tip capicola beef t-bone shank tenderloin turducken ball tip salami pork belly shankle. Kielbasa pastrami brisket, kevin spare ribs swine tail beef jerky venison filet mignon. Kevin leberkas ball tip, brisket bresaola chuck meatloaf beef doner drumstick hamburger capicola chicken. Tri-tip biltong drumstick pork prosciutto strip steak pastrami brisket shank hamburger flank tail cow. Pastrami beef ribs ribeye boudin spare ribs pork loin. Meatloaf tail pork belly strip steak doner. T-bone meatball pastrami, pork strip steak salami tail beef boudin leberkas. Venison t-bone fatback, pig brisket pork loin landjaeger turkey tri-tip biltong. Drumstick tri-tip hamburger boudin meatball pork pork chop short ribs chuck doner t-bone bacon frankfurter porchetta beef. Turkey cow meatball andouille pancetta, flank strip steak ham hock. Frankfurter corned beef rump turducken brisket, jerky short loin flank tri-tip ball tip ham hock swine spare ribs.";


	  /*** Auto Layout ***/

    NSDictionary *views = NSDictionaryOfVariableBindings(scrollView, scrollViewLabel);

    NSArray *scrollViewLabelConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"H:|[scrollViewLabel(scrollView)]" options:0 metrics:nil views:views];
    [scrollView addConstraints:scrollViewLabelConstraints];

    scrollViewLabelConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"V:|[scrollViewLabel]|" options:0 metrics:nil views:views];
    [scrollView addConstraints:scrollViewLabelConstraints];

    NSArray *scrollViewConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"H:|-[scrollView]-|" options:0 metrics:nil views:views];
    [self.view addConstraints:scrollViewConstraints];

    scrollViewConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"V:|-[scrollView]-|" options:0 metrics:nil views:views];
    [self.view addConstraints:scrollViewConstraints];

}

```

## Explanation

### `@"H:|[scrollViewLabel(scrollView)]"`

This means: "The width of the label must be equal to the scrollview's width".

In AutoLayout, the `contentSize` of a scroll view is determined by the size of its subviews. So because the label is the scrollview's only subview, that constraint also implicitly means "Set the width of the scrollview's `contentsize` to the width of the label".

### `@"V:|[scrollViewLabel]|"`

This means: "The height of the scrollview label must take up all the vertical space it needs within its superview. The scrollview's `contentSize`'s height must also be equal to it".

## References aka Further Reading


### Articles

* "Dynamic UILabel height inside UIScrollView with Autolayout" question on Stackoverflow. [Link](http://stackoverflow.com/questions/19192141/dynamic-uilabel-height-inside-uiscrollview-with-autolayout)


### Documentation

* "Apple Technical Note TN2154: UIScrollView And Autolayout". *iOS Developer Library.* [Link](https://developer.apple.com/library/ios/technotes/tn2154/_index.html)

### Filler Text

* "Bacon Ipsum: A Meatier Lorem Ipsum Generator". [Link](http://baconipsum.com/)
