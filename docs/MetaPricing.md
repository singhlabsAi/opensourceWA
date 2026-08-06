## Meta WhatsApp Business pricing explained

I’m assuming you mean the **WhatsApp Business Platform/API**, used for bulk broadcasts, approved templates, chatbots, CRM integrations and automation—not the ordinary WhatsApp Business mobile app.

The key idea is:

> **Creating or approving a template normally has no Meta fee. You pay when an eligible message is successfully delivered to a customer.**

Since July 1, 2025, Meta has primarily used **per-delivered-message pricing**, rather than charging one price for an entire 24-hour conversation. The applicable rate depends on the recipient’s country and the message category. ([WhatsApp for Business][1])

## 1. Is there a template creation cost?

**Meta does not normally charge you to:**

* Create a message template
* Submit it for approval
* Edit or manage templates
* Keep an approved template in your account
* Send a template that fails and is never delivered

The cost arises when an approved template message is **delivered**.

However, your WhatsApp provider—such as Interakt, WATI, AiSensy, Gupshup, Twilio or another BSP—may separately charge for its software, template-management interface, campaign tools or onboarding.

So “template cost” can mean two different things:

1. **Meta template fee:** Normally no creation fee; delivery is charged.
2. **Provider/platform fee:** Monthly subscription, markup or campaign fee imposed by your BSP.

## 2. The four message categories

### Marketing messages

These include:

* Promotional broadcasts
* Offers and discount announcements
* New product launches
* Sale reminders
* Abandoned-cart reminders
* Cross-selling and upselling
* “We miss you” or re-engagement messages
* Event promotions
* Informational messages containing promotional content

Marketing is normally the **most expensive category**.

Even when a customer has recently messaged you, a marketing template can still be chargeable unless a specific free-entry-point exception applies.

### Utility messages

These must be related to a particular transaction, account, request or customer action, for example:

* Order confirmation
* Shipping update
* Delivery notification
* Appointment reminder
* Payment confirmation
* Invoice
* Refund update
* Account-related notification requested by the customer

Utility templates must not contain promotional content. Adding something like “Buy again and get 20% off” may cause Meta to classify the template as marketing.

Meta states that utility messages sent in response to users can be free in qualifying circumstances. ([WhatsApp for Business][1])

### Authentication messages

These are identity-verification messages such as:

* One-time passwords
* Login codes
* Account-verification codes
* Transaction-verification codes
* Password-recovery codes

Authentication pricing can also have a separate **authentication-international** rate where the recipient and business geography meet Meta’s international-authentication criteria.

### Service messages

These are normal replies to a customer who has initiated the conversation.

When a customer messages your business, a **24-hour customer-service window** opens. During that window, you can respond with normal non-template service messages. Meta’s current public pricing page says these service messages are not charged, and every new customer message resets that 24-hour window. ([WhatsApp for Business][1])

Pricing policies can change, so your WhatsApp Manager billing screen and Meta’s live rate card should be treated as the final source.

## 3. How broadcast pricing works

A broadcast is **not charged as one campaign**.

It is calculated recipient by recipient and delivered message by delivered message.

Suppose you send one marketing template to 10,000 Indian numbers:

* 10,000 messages submitted
* 9,200 successfully delivered
* 500 failed
* 300 remained undelivered

Meta generally charges for the **9,200 delivered messages**, not all 10,000 submissions.

If your rate were ₹0.8631 per delivered marketing message:

**9,200 × ₹0.8631 = ₹7,940.52 Meta message cost**

Then add, where applicable:

* BSP markup
* Platform subscription
* Campaign fee
* Automation or chatbot fee
* Taxes such as GST
* Payment-processing or currency-conversion charges

Meta explicitly states that billing is based on delivered messages, the recipient’s market and the message category. ([WhatsApp for Business][2])

## 4. Approximate India rates

Current rates must be checked in Meta’s live rate card because Meta revises pricing periodically and volume discounts may apply. Recent 2026 India rate references commonly show approximately:

| Category                                   |                                                      Indicative Meta charge |
| ------------------------------------------ | --------------------------------------------------------------------------: |
| Marketing                                  |                                        Around ₹0.8631 per delivered message |
| Utility                                    | Around ₹0.1150 per delivered message before applicable discounts/free rules |
| Authentication                             |            Around ₹0.1150 per delivered message before applicable discounts |
| Service reply in qualifying 24-hour window |                            ₹0 Meta charge under the current published model |
| Authentication international               |                                             Separate, generally higher rate |

These are **Meta charges only**, normally before GST and provider fees. Meta’s official site says rates vary by market and category, and utility and authentication pricing can receive volume-tier discounts. ([WhatsApp for Business][2])

Because India pricing has changed before, do not build a long-term budget using a screenshot or an old blog post. Use the live India/INR rate shown in your WhatsApp Manager or Meta rate card on the billing date.

## 5. What does “charged per message” really mean?

Imagine you send three messages to the same customer:

1. Marketing template: “Our sale has started.”
2. Marketing template: “Here is your coupon.”
3. Marketing template: “Last chance to buy.”

Under per-message charging, all three successfully delivered marketing templates can be charged separately:

**3 delivered marketing messages × marketing rate**

It is no longer enough to assume that opening one paid conversation lets you send unlimited templates during the following 24 hours.

This is why multiple campaign follow-ups can become expensive.

## 6. Mixed-category campaign example

You contact 5,000 Indian customers:

* 5,000 marketing offers
* 2,000 order confirmations
* 1,000 OTPs
* 3,000 free customer-support replies

Using illustrative rates:

* Marketing: 5,000 × ₹0.8631 = **₹4,315.50**
* Utility: 2,000 × ₹0.115 = **₹230**
* Authentication: 1,000 × ₹0.115 = **₹115**
* Qualifying service replies: **₹0**

**Illustrative Meta total: ₹4,660.50**

With 18% GST, where applicable:

**₹4,660.50 × 18% = ₹838.89 GST**

**Total after GST: ₹5,499.39**

Then add your BSP’s charges.

This example assumes everything was delivered and that no volume discounts, free-entry-point rules or free utility conditions applied.

## 7. The 24-hour customer-service window

The window begins when the customer messages you.

Example:

* Customer messages at Monday, 2:00 p.m.
* You can send normal customer-service replies until Tuesday, 2:00 p.m.
* Customer sends another message Tuesday at 10:00 a.m.
* The window resets and now runs until Wednesday, 10:00 a.m.

Inside the open window, you can generally send free-form service replies without using an approved template.

Outside the window, you normally need an approved template to restart business-initiated communication.

Important distinction:

* A genuine support reply can be a service message.
* A sale, offer or promotional follow-up remains marketing in nature.
* Opening the customer-service window does not automatically turn promotional content into free service messaging.

## 8. The 72-hour free entry-point window

A valuable exception applies when a customer enters WhatsApp through certain Meta-owned entry points, such as:

* A click-to-WhatsApp advertisement
* A qualifying Facebook Page call-to-action button

When the customer starts the conversation through a qualifying entry point, Meta states that messages may be free for the following **72 hours**. ([WhatsApp for Business][1])

This does **not** mean the advertisement itself is free. You still pay Meta Ads charges for the Facebook or Instagram campaign. It means eligible WhatsApp message charges during the 72-hour window can be waived.

## 9. Utility and authentication volume discounts

Meta uses volume tiers for utility and authentication messages.

This normally works progressively:

* Messages in the first tier receive the first-tier rate.
* Only messages falling into the next tier receive the next-tier discount.
* Crossing a threshold does not necessarily make every earlier message cheaper retroactively.

Marketing messages generally do not receive the same type of volume-tier treatment described for utility and authentication.

Meta confirms that utility and authentication messages can unlock lower rates as message volume grows. ([WhatsApp for Business][2])

## 10. How Meta decides the recipient’s country

The rate is generally based on the **recipient’s WhatsApp phone-number market**, not simply where your business is located.

Therefore, an Indian company sending to:

* An Indian number: India rate
* A UAE number: UAE rate
* A UK number: UK rate
* A US/Canada number: North America rate

A single broadcast containing numbers from multiple countries can therefore have different rates within the same campaign.

## 11. Who decides the template category?

You choose a category when submitting the template, but Meta reviews and may reclassify it.

Common causes of reclassification to marketing include:

* Discounts or offers
* Product recommendations
* Promotional buttons
* Cross-selling language
* Generic engagement messages
* Requests to visit a store or website
* Utility information combined with advertising

Example:

**Utility:**
“Your order 4567 has been dispatched and will arrive tomorrow.”

**Likely marketing:**
“Your order 4567 has been dispatched. Buy another product today and get 20% off.”

The promotional addition can cause the complete message to be treated as marketing.

## 12. Meta fees versus BSP fees

Your complete WhatsApp bill may contain several layers.

### Meta message charges

Charged according to:

* Delivered-message count
* Message category
* Recipient market
* Applicable volume tier
* Free-window eligibility

### BSP or software-platform charges

Your provider may use one or more of these models:

* Fixed monthly subscription
* Annual plan
* Per-user or per-agent fee
* Per-message markup
* Campaign charge
* Contact-count charge
* Automation-flow charge
* Chatbot-session charge
* Onboarding or setup fee
* Number-hosting fee
* Integration/API fee

### Taxes

For Indian businesses, the invoice may include GST, depending on the billing entity and arrangement.

This is why two providers using the same Meta infrastructure can quote very different final prices.

## 13. Common provider pricing structures

### Zero-markup model

You pay:

**Meta charges + fixed monthly software fee + tax**

This is usually transparent for businesses sending high volume.

### Per-message markup model

You pay:

**Meta rate + provider markup for every message + tax**

Example:

* Meta marketing rate: ₹0.8631
* Provider markup: ₹0.15
* Effective pre-tax rate: ₹1.0131 per message

### Bundled model

The provider quotes something like:

* ₹2,999 monthly
* 5,000 messages included
* Extra messages at a stated rate

You must check whether the “included messages” cover:

* Only the platform fee
* Meta fees as well
* Only utility messages
* Any message category
* Delivered or submitted messages

### Wallet/prepaid model

You add money to a wallet and deductions happen when messages are delivered. Ask whether the displayed deduction combines Meta fees, markup and GST.

## 14. Does one template sent to 10,000 users count as one message?

No.

It counts as up to **10,000 separately delivered messages**.

Formula:

**Broadcast cost = successfully delivered recipients × applicable rate for each recipient**

If recipients belong to different countries, calculate each country separately.

## 15. What happens with images, PDFs and videos?

Meta pricing is principally based on category and delivery, not simply on whether the template contains:

* Text
* Image
* Video
* PDF
* Buttons
* Quick replies

Nevertheless, your BSP may impose media-storage, bandwidth or campaign-related charges.

A marketing template with an image is still billed as marketing. A utility template with an invoice PDF is still billed according to utility rules.

## 16. Are failed, blocked or unread messages charged?

The main charging event is **delivery**, not whether the customer opens or reads the message.

Generally:

* Submitted but failed: not delivered, so no Meta delivery charge
* Invalid WhatsApp number: no delivery
* Customer blocked the business: likely no delivery
* Delivered but not read: charge can still apply
* Delivered and read: charge applies
* Customer ignores the message: charge still applies if delivered

Your BSP may still count API attempts differently for its own plan, so check its contract.

## 17. Templates and customer consent

Paying Meta does not give permission to message anyone.

For proactive broadcasts, you should have valid WhatsApp opt-in and comply with:

* WhatsApp Business Messaging Policy
* Applicable privacy and anti-spam rules
* Clear unsubscribe or opt-out handling
* Appropriate message frequency
* Accurate template categorisation

Poor-quality campaigns can result in:

* Customers blocking or reporting you
* Reduced messaging limits
* Template pausing or disabling
* Lower quality rating
* Restrictions on your WhatsApp Business Account

## 18. A practical monthly-budget formula

Use:

**Monthly WhatsApp cost = Meta message charges + BSP subscription + BSP markups + integrations/agents + taxes**

For campaigns:

**Meta campaign cost = Σ(delivered messages in each country and category × that country-category rate)**

Example budget sheet:

| Component                    | Calculation                                                 |
| ---------------------------- | ----------------------------------------------------------- |
| Marketing                    | Delivered marketing messages × marketing rate               |
| Utility                      | Chargeable delivered utility messages × tiered utility rate |
| Authentication               | Delivered OTP messages × tiered authentication rate         |
| International authentication | Applicable delivered messages × international rate          |
| Service                      | Chargeable service messages under current rules             |
| BSP fee                      | Monthly plan + agent/add-on fees                            |
| Markup                       | Delivered messages × BSP markup                             |
| Tax                          | Applicable percentage on taxable invoice value              |

## 19. Questions to ask any WhatsApp provider before purchasing

Ask the provider to state in writing:

1. Are Meta’s charges passed through at cost?
2. Is there any per-message markup?
3. Is the monthly plan separate from Meta fees?
4. Are rates before or after GST?
5. Are charges based on submitted or delivered messages?
6. Are all message categories priced separately?
7. Are utility and authentication volume discounts passed to you?
8. Who owns the WhatsApp Business Account and phone number?
9. Can you move the number to another BSP later?
10. Are chatbot, agent, automation and integration fees extra?

The most important protection is to require the quotation to separate:

> **Meta fee | Provider fee | Tax | Optional add-ons**

## Bottom line

For a normal promotional broadcast in India, your cost is approximately:

**Number of successfully delivered recipients × current India marketing rate**

Creating the template itself normally does not cost anything. A message can then become more expensive because of provider markup, subscription fees and GST.

Customer-service replies within the qualifying 24-hour window are currently listed by Meta as free, while click-to-WhatsApp ads can create a 72-hour free messaging window. Utility and authentication messages are cheaper than marketing and can qualify for volume discounts. ([WhatsApp for Business][1])

[1]: https://business.whatsapp.com/products/platform-pricing "WhatsApp Business Platform Pricing | WhatsApp API Pricing"
[2]: https://business.whatsapp.com/products/platform-pricing?utm_source=chatgpt.com "WhatsApp Business Platform Pricing"
