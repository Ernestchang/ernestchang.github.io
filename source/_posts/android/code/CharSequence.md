---
title: 类似网页超链接实现
date: 

categories: 
- android 
- code
tags:  
- android
- code
---

```
CharSequence agreementTxt = (getString(R.string.maimai_register_license_html) + "《" + getString(R.string.app_name) + getString(R.string.maimai_license_html) + "》");
        SpannableString content = new SpannableString(agreementTxt);
        content.setSpan(new ClickableSpan() {

            @Override
            public void onClick(View widget) {
                L.d("into click");
                String url = "http://maimai.cn/maimai_license";

                new FinestWebView.Builder(RegisterActivity.this).theme(R.style.FinestWebViewTheme)
//                   .titleDefault("Vimeo")
                        .showUrl(false)
                        .statusBarColorRes(R.color.bg_top_tab)
                        .toolbarColorRes(R.color.bg_top_tab)
                        .titleColorRes(R.color.finestWhite)
                        .iconDefaultColorRes(R.color.finestWhite)
                        .progressBarColorRes(R.color.finestWhite)
                        .stringResCopiedToClipboard(R.string.copied_to_clipboard)
                        .showSwipeRefreshLayout(false)
                        .menuSelector(R.drawable.selector_light_theme)
                        .menuTextGravity(Gravity.CENTER)
                        .menuTextPaddingRightRes(R.dimen.defaultMenuTextPaddingLeft)
                        .dividerHeight(0)
                        .gradientDivider(false)
                        .setCustomAnimations(R.anim.slide_up, R.anim.hold, R.anim.hold, R.anim.slide_down)
                        .show(url);

                WebView webView  = new WebView(JinkeApplication.getInstance());


            }

            @Override
            public void updateDrawState(TextPaint ds) {
                super.updateDrawState(ds);
                ds.setUnderlineText(false);
                ds.setColor(getResources().getColor(android.R.color.holo_blue_dark));
                mobileRegisterLicenseTips.setHighlightColor(getResources().getColor(android.R.color.transparent));
                ds.clearShadowLayer();
            }
        }, agreementTxt.length() - 10, agreementTxt.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        mobileRegisterLicenseTips.setText(content);
        mobileRegisterLicenseTips.setMovementMethod(LinkMovementMethod.getInstance());
```

