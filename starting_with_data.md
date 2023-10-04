# Question 1:

Show the state of the stock level compared to the ordered quantity our company is expected to fulfill, of every product.  Show the amount of surplus inventory we have (stocklevel-orderedquantity) for each product, and an indication of status (how urgently do we need to re-order, or be concerned about running into issues fulfilling to our customers?).  Rank the products according to the urgency with which we are out of stock, so the purchasing department knows their priorities - i.e. which products they should work most immediately on (Rank #1) to source more quantity in order to meet our fulfillment obligations.


### SQL Queries:
```SQL
-- Build result set with rank on purchasing urgency based on absolute value/size of the negative surplus quantity, using a window function

SELECT	sku,
		name,
		orderedquantity,
		stocklevel,
		(stocklevel-orderedquantity) AS surplus_quantity,
		CASE
			WHEN (stocklevel-orderedquantity) > 0 THEN 'GREEN: OK for fulfill'
			WHEN (stocklevel-orderedquantity) = 0 THEN 'YELLOW: Next Order Trouble'
			ELSE 'RED: Stock Up Immediately'
		END AS purchasing_status,
		DENSE_RANK() OVER (ORDER BY (stocklevel-orderedquantity) ASC) AS rank_purchasing_urgency
FROM products_clean p
ORDER BY (stocklevel-orderedquantity) ASC
```

## Answer:

The top 10 priorities for the purchasing department to work on are these products, which are all significantly under stock (more have been ordered than we have on hand to fulfill):

- "GGOEGFSR022099": "Kick Ball"
- "GGOEGOLC014299": "Metallic Notebook Set"
- "GGOEGGOA017399": "Maze Pen"
- "GGOEGBJC019999": "Collapsible Shopping Bag"
- "GGOEGKAA019299": "Switch Tone Color Crayon Pen"
- "GGOENEBQ079099": "Protect Smoke + CO White Battery Alarm-USA"
- "GGOEGDHQ015399": "26 oz Double Wall Insulated Bottle"
- "GGOEGHGT019599": "Sunglasses"
- "GGOEYOCR077399": "RFID Journal"
- "GGOEGFKA022299": "Keyboard DOT Sticker"

The entire result set is:
| sku              | name                                                | orderedquantity | stocklevel | surplus_quantity | purchasing_status          | rank_purchasing_urgency |
|------------------|-----------------------------------------------------|-----------------|------------|------------------|----------------------------|-------------------------|
| GGOEGFSR022099   | Kick Ball                                           | 15170           | 723        | -14447           | RED: Stock Up Immediately  | 1                       |
| GGOEGOLC014299   | Metallic Notebook Set                               | 2718            | 610        | -2108            | RED: Stock Up Immediately  | 2                       |
| GGOEGGOA017399   | Maze Pen                                            | 1748            | 324        | -1424            | RED: Stock Up Immediately  | 3                       |
| GGOEGBJC019999   | Collapsible Shopping Bag                            | 1184            | 117        | -1067            | RED: Stock Up Immediately  | 4                       |
| GGOEGKAA019299   | Switch Tone Color Crayon Pen                        | 1163            | 388        | -775             | RED: Stock Up Immediately  | 5                       |
| GGOENEBQ079099   | Protect Smoke + CO White Battery Alarm-USA          | 999             | 325        | -674             | RED: Stock Up Immediately  | 6                       |
| GGOEGDHQ015399   | 26 oz Double Wall Insulated Bottle                  | 845             | 256        | -589             | RED: Stock Up Immediately  | 7                       |
| GGOEGHGT019599   | Sunglasses                                          | 887             | 376        | -511             | RED: Stock Up Immediately  | 8                       |
| GGOEYOCR077399   | RFID Journal                                        | 935             | 433        | -502             | RED: Stock Up Immediately  | 9                       |
| GGOEGFKA022299   | Keyboard DOT Sticker                                | 480             | 189        | -291             | RED: Stock Up Immediately  | 10                      |
| GGOEGOCT019199   | Red Spiral  Notebook                                | 316             | 43         | -273             | RED: Stock Up Immediately  | 11                      |
| GGOEGDHC015299   | 23 oz Wide Mouth Sport Bottle                       | 402             | 142        | -260             | RED: Stock Up Immediately  | 12                      |
| GGOEGAWQ062948   | Baby Essentials Set                                 | 261             | 77         | -184             | RED: Stock Up Immediately  | 13                      |
| GGOEGETR014599   | Tube Power Bank                                     | 218             | 72         | -146             | RED: Stock Up Immediately  | 14                      |
| GGOEGHPB003410   | Snapback Hat Black                                  | 230             | 105        | -125             | RED: Stock Up Immediately  | 15                      |
| GGOEYAEJ029517   | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182722          | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180759          | Lunch Bag                                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180814          | Micro Wireless Earbud                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180850          | Ballpoint Stylus Pen                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183118          | Women's Fleece Hoodie Black                         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182806          | Onesie Green                                        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEC029117   | Women's Short Sleeve Hero Tee Sky Blue              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183241          | 25L Classic Rucksack                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183203          | Device Holder Sticky Pad                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184697          | Mood Original Window Decal                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183225          | Luggage Tag                                         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182723          | Men's 100% Cotton Short Sleeve Hero Tee White       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182745          | Men's Quilted Insulated Vest Black                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182720          | Men's Short Sleeve Badge Tee Charcoal               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0581     | Women's Colorblock Tee White                        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182741          | Men's Airflow 1/4 Zip Pullover Lapis                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182772          | Women's Performance Full Zip Jacket Black           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182755          | Men's Colorblock Tee White/Heather                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEA030614   | Women's Recycled Fabric Tee                         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184720          | Women's Short Sleeve Hero Tee Sky Blue              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182761          | Women's Yoga Jacket Black                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184689          | Men's Short Sleeve Tee                              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183239          | 25L Classic Rucksack                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAWT061750   | Android Onesie Gold                                 | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183229          | Cam Outdoor Security Camera - USA                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184682          | Mobile Phone Vent Mount                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183234          | Learning Thermostat 3rd Gen-USA - Stainless Steel   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAEJ029614   | Women's Short Sleeve Tri-blend Badge Tee Grey       | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182859          | Toddler Raglan Shirt Blue Heather/Navy              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182863          | Toddler Sports T-shirt Red                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180811          | Plastic Sliding Flashlight                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182717          | Men's Short Sleeve Hero Tee Light Blue              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182599          | Women's Long Sleeve Tee Lavender                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182750          | Men's Performance 1/4 Zip Pullover Heather/Black    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183188          | High Capacity 10,400mAh Charger                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183205          | Zipper-front Sports Bag                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ035317   | Vintage Henley Grey/Black                           | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182980          | Pet Feeding Mat                                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184707          | Women's Typography Short Sleeve Tee                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALJ060113   | Women's Short Sleeve Performance Tee Pewter         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATB060417   | Women's Quilted Insulated Vest Black                | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180813          | Tube Power Bank                                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183219          | Leather Journal                                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183212          | Android RFID Journal                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182778          | Women's Softshell Jacket Black/Grey                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184683          | Women's Short Sleeve Tee                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183196          | 4400mAh Power Bank                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180769          | Laptop Backpack                                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183222          | Leather Perforated Journal                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182774          | Women's Yoga Pants                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183220          | Leather Journal-Brown                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182780          | Women's Shell Jacket Blue/Black                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180762          | Tote Bag                                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182179          | Android Women's Short Sleeve Badge Tee Dark Heather | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0734     | Men's Short Sleeve Hero Tee Heather                 | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182816          | Infant Zip Hood Royal Blue                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182997          | 17oz Stainless Steel Sport Bottle                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0324     | Android Men's Short Sleeve Tri-blend Hero Tee Grey  | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAXXX0833     | Android Men's Paradise Short Sleeve Tee Olive       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0575     | Women's Weatherblock Shell Jacket Black             | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0592     | Men's Airflow 1/4 Zip Pullover Black                | 8               | 8          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180770          | Yoga Mat Blue                                       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWAAJ082815   | Men's Short Sleeve Tee                              | 5               | 5          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ029916   | Women's V-Neck Tee Charcoal                         | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184681          | 17 oz Double Wall Stainless Steel Insulated Bottle  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADB056613   | Men's Weatherblock Shell Jacket Black               | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPB058215   | Women's Lightweight Microfleece Jacket              | 5               | 5          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAYC068726   | Android Youth Short Sleeve T-shirt Aqua             | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183086          | Youth Girl Tee Green                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181573          | Snapback Hat Black                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180757          | Yoga Block                                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182812          | Infant Zip Hood Pink                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAWJ062548   | Android Infant Short Sleeve Tee Pewter              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAQB073217   | Women's Fleece Hoodie Black                         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ028915   | Women's Short Sleeve Hero Dark Grey                 | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ031317   | Tri-blend Hoodie Grey                               | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAH034013   | Men's Vintage Badge Tee Sage                        | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAEJ029015   | Women's Short Sleeve Hero Tee Charcoal              | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181019          | Tri-blend Hoodie Grey                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182714          | Men's Long Sleeve Raglan Ocean Blue                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEB084517   | BLM Sweatshirt                                      | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182553          | Men's Vintage Henley                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADJ057117   | Men's Performance 1/4 Zip Pullover Heather/Black    | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0314     | Men's 3/4 Sleeve Henley                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180764          | Canvas Tote Natural/Navy                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAXC065255   | Toddler Short Sleeve T-shirt Royal Blue             | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAQ057216   | Men's Colorblock Tee White/Heather                  | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182739          | Men's Watershed Full Zip Hoodie Grey                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPB058212   | Women's Lightweight Microfleece Jacket              | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182793          | Men's Long & Lean Tee Charcoal                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183214          | Hard Cover Journal                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAQB036017   | Women's Fleece Hoodie                               | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182719          | Men's Short Sleeve Hero Tee Charcoal                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184733          | Learning Thermostat 3rd Gen-USA - White             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWAAJ083513   | Men's Typography Short Sleeve Tee                   | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180844          | Gunmetal Roller Ball Pen                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182764          | Women's Convertible Vest-Jacket Black               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182740          | Men's Airflow 1/4 Zip Pullover Black                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAC032117   | Men's Short Sleeve Hero Tee Light Blue              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182828          | Toddler Short Sleeve T-shirt Royal Blue             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAYB068056   | Youth Baseball Raglan Heather/Black                 | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182751          | Men's Performance Full Zip Jacket Black             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184695          | Baby on Board Window Decal                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYHPA003610   | Wool Heather Cap Heather/Black                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181138          | Rucksack                                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182581          | Women's Fleece Hoodie                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAWT061754   | Android Onesie Gold                                 | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAB058918   | Men's Short Sleeve Performance Badge Tee Black      | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182771          | Women's 1/4 Zip Jacket Charcoal                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182743          | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180838          | Metal Texture Roller Pen                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182569          | Men's  Zip Hoodie                                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALP036316   | Women's Long Sleeve Tee Lavender                    | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAXR065155   | Toddler Short Sleeve Tee Red                        | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0567     | Men's Softshell Jacket Black/Grey                   | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ028916   | Women's Short Sleeve Hero Dark Grey                 | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182737          | Women's 3/4 Sleeve Baseball Raglan Heather/Black    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWXXX0835     | Men's Typography Short Sleeve Tee                   | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYHPA003510   | Trucker Hat                                         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAAJ032517   | Men's Short Sleeve Hero Tee Charcoal                | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184721          | BLM Sweatshirt                                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182848          | Toddler Hoodie Royal Blue                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYDHJ019399   | 24 oz  Sergeant Stripe Bottle                       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADJ059815   | Men's Convertible Vest-Jacket Pewter                | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182898          | Android Youth Short Sleeve T-shirt Aqua             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183112          | Men's Fleece Hoodie Black                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180779          | 1 oz Hand Sanitizer                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184734          | Learning Thermostat 3rd Gen-USA - Copper            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183096          | Youth Sports Tee Navy                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184708          | Women's Typography Short Sleeve Tee                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184675          | UpCycled Bike Saddle Bag                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWXXX0828     | Men's Short Sleeve Tee                              | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180766          | Sport Bag                                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181572          | Android Wool Heather Cap Heather/Black              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182705          | Android Men's Long & Lean Badge Tee Charcoal        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182785          | Women's Lightweight Microfleece Jacket              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180767          | Slim Utility Travel Bag                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182502          | Women's Yoga Jacket Black                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180793          | 26 oz Double Wall Insulated Bottle                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180820          | Doodle Decal                                        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183186          | G Noise-reducing Bluetooth Headphones               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180905          | Men's Long Sleeve Raglan Ocean Blue                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183152          | Youth Short Sleeve Tee Red                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182777          | Women's Performance Golf Polo Blue                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182658          | Women's Short Sleeve Hero Tee Sky Blue              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADC059313   | Men's Airflow 1/4 Zip Pullover Lapis                | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182742          | Men's Convertible Vest-Jacket Pewter                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0733     | Women's Short Sleeve Hero Tee Heather               | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGHPA002910   | Trucker Hat                                         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181151          | Gift Card - $50.00                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAWC061949   | Infant Short Sleeve Tee Royal Blue                  | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180782          | Seat Pack Organizer                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182725          | Men's Long & Lean Tee Grey                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADB056618   | Men's Weatherblock Shell Jacket Black               | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183226          | Android Luggage Tag                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAAB081016   | Android Men's Outerstellar Short Sleeve Tee Black   | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183228          | Cam Indoor Security Camera - USA                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182999          | Bongo Cupholder Bluetooth Speaker                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181664          | Waterproof Gear Bag                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATB060414   | Women's Quilted Insulated Vest Black                | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182909          | Collapsible Pet Bowl                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0601     | Women's Short Sleeve Performance Tee Pewter         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182559          | Android Men's Vintage Henley                        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181150          | Gift Card - $250.00                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184705          | Women's Typography Short Sleeve Tee                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182784          | Women's Short Sleeve Hero Tee White                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180840          | Satin Black Ballpoint Pen                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAEJ029615   | Women's Short Sleeve Tri-blend Badge Tee Grey       | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGHPJ080010   | French Terry Cap                                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182995          | Car Clip Phone Holder                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182752          | Men's Short Sleeve Performance Badge Tee Charcoal   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182575          | Android Men's  Zip Hoodie                           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0349     | Android BTTF Moonshot Graphic Tee                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0582     | Women's Lightweight Microfleece Jacket              | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0577     | Women's Performance Golf Polo Blue                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182781          | Women's Short Sleeve Hero Tee Black                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182593          | Men's Pullover Hoodie Grey                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182738          | Women's Long Sleeve Blended Cardigan Charcoal       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182383          | Men's Watershed Full Zip Hoodie Grey                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183230          | Protect Smoke + CO White Battery Alarm-USA          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180756          | Windup Android                                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182587          | Android Women's Fleece Hoodie                       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183000          | Pocket Bluetooth Speaker                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0346     | Men's Vintage Tee                                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182721          | Men's 100% Cotton Short Sleeve Hero Tee Black       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182873          | Android Toddler Short Sleeve T-shirt Pink           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183240          | 25L Classic Rucksack                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0730     | Men's Performance Hero Tee Gunmetal                 | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183194          | Executive Umbrella                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0572     | Men's Colorblock Tee White/Heather                  | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180868          | Crunch Noise Dog Toy                                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWXXX0827     | Women's Short Sleeve Tee                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0598     | Men's Convertible Vest-Jacket Pewter                | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180808          | Basecamp Explorer Powerbank Flashlight              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184676          | UpCycled Handlebar Bag                              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPB058217   | Women's Lightweight Microfleece Jacket              | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAYR068225   | Youth Short Sleeve Tee Red                          | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADB056615   | Men's Weatherblock Shell Jacket Black               | 12              | 12         | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALJ072913   | Women's Performance Hero Tee Gunmetal               | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAEB028314   | Android Women's Short Sleeve Hero Tee Black         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181149          | Gift Card - $25.00                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWAAJ082817   | Men's Short Sleeve Tee                              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAWR061849   | Infant Short Sleeve Tee Red                         | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATJ060515   | Women's Quilted Insulated Vest White                | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182706          | Android Stretch Fit Hat                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180862          | Screen Cleaning Cloth                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAXXX0811     | Android Men's Pep Rally Short Sleeve Tee Navy       | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183106          | Women's Performance Hero Tee Gunmetal               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0335     | Men's Long Sleeve Pullover Badge Tee Heather        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAXR065130   | Toddler Short Sleeve Tee Red                        | 12              | 12         | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182701          | Android Men's Long Sleeve Badge Crew Tee Heather    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADB059213   | Men's Airflow 1/4 Zip Pullover Black                | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPB057516   | Women's Weatherblock Shell Jacket Black             | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAYH067924   | Youth Girl Tee Green                                | 9               | 9          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183210          | RFID Journal                                        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182788          | Yoga Mat                                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0576     | Women's Softshell Jacket Black/Grey                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181139          | Waterproof Backpack                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYAEB035617   | Men's Vintage Tank                                  | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180792          | 20 oz Stainless Steel Insulated Tumbler             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAFJ036217   | Men's Pullover Hoodie Grey                          | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182753          | Men's Weatherblock Shell Jacket Black               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183213          | Android Hard Cover Journal                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183221          | Leather Journal-Black                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEYOBR078599   | Luggage Tag                                         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWAAJ082814   | Men's Short Sleeve Tee                              | 6               | 6          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAWR061049   | Onesie Red/Graphite                                 | 10              | 10         | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0660     | Toddler Sports T-shirt Red                          | 10              | 10         | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAWR061148   | Onesie Red                                          | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAEJ028213   | Android Women's Short Sleeve Badge Tee Dark Heather | 6               | 6          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPC058812   | Women's Shell Jacket Blue/Black                     | 7               | 7          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGHPA003010   | Wool Heather Cap Heather/Navy                       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180809          | Flashlight                                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182176          | Android Women's Short Sleeve Badge Tee Dark Heather | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAXR066055   | Toddler Sports T-shirt Red                          | 5               | 5          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAEB028316   | Android Women's Short Sleeve Hero Tee Black         | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182765          | Women's Convertible Vest-Jacket Sea Foam Green      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWAAJ082813   | Men's Short Sleeve Tee                              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180771          | Yoga Mat Purple                                     | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182820          | Baby Essentials Set                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATH060717   | Women's Convertible Vest-Jacket Sea Foam Green      | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184698          | Mood Ninja Window Decal                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGGCX056499   | Gift Card - $50.00                                  | 9               | 9          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAAB034814   | Android BTTF Cosmos Graphic Tee                     | 6               | 6          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADC059516   | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPC058814   | Women's Shell Jacket Blue/Black                     | 4               | 4          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALJ072915   | Women's Performance Hero Tee Gunmetal               | 6               | 6          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183223          | Spiral Leather Journal                              | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182726          | Men's Long & Lean Tee Charcoal                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATB060416   | Women's Quilted Insulated Vest Black                | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183015          | Onesie Heather                                      | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182605          | Women's Tee Grey                                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEWXXX0834     | Women's Typography Short Sleeve Tee                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180754          | 8 pc Android Sticker Sheet                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0729     | Women's Performance Hero Tee Gunmetal               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180833          | Rubber Grip Ballpoint Pen 4 Pack                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEJ029915   | Women's V-Neck Tee Charcoal                         | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALJ073317   | Women's Short Sleeve Hero Tee Heather               | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGALJ073313   | Women's Short Sleeve Hero Tee Heather               | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182709          | Android Women's Long Sleeve Blended Cardigan Grey   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAEB084514   | BLM Sweatshirt                                      | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180781          | Suitcase Organizer Cubes                            | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182769          | Women's Short Sleeve Performance Tee Pewter         | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181148          | Gift Card- $100.00                                  | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182173          | Stylus Pen w/ LED Light                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGATC060316   | Women's 1/4 Zip Performance Pullover Two-Tone Blue  | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184620          | Men's Performance Full Zip Jacket Black             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180748          | Android Lunch Kit                                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGADJ059717   | Men's Quilted Insulated Vest Battleship Grey        | 3               | 3          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182754          | Men's Softshell Jacket Black/Grey                   | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9181137          | Alpine Style Backpack                               | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9184737          | Pack of 9 Decal Set                                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182746          | Men's Quilted Insulated Vest Battleship Grey        | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAWJ062550   | Android Infant Short Sleeve Tee Pewter              | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGDHB072099   | Insulated Stainless Steel Bottle                    | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAPB057517   | Women's Weatherblock Shell Jacket Black             | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180806          | Clip-on Compact Charger                             | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEAAEH035216   | Android Men's Vintage Henley                        | 2               | 2          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9180824          | Foam Can and Bottle Cooler                          | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAER035517   | Men's Vintage Tank                                  | 1               | 1          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182773          | Women's Short Sleeve Performance Tee Charcoal       | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9183091          | Youth Baseball Raglan Heather/Black                 | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| 9182903          | Android Youth Short Sleeve T-shirt Pewter           | 0               | 0          | 0                | YELLOW: Next Order Trouble | 16                      |
| GGOEGAAX0684     | Youth Short Sleeve T-shirt Green                    | 11              | 12         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAXJ066229   | Android Toddler Short Sleeve T-shirt Pewter         | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAB058915   | Men's Short Sleeve Performance Badge Tee Black      | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALJ060115   | Women's Short Sleeve Performance Tee Pewter         | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAYR068626   | Youth Short Sleeve Tee Red                          | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPB057514   | Women's Weatherblock Shell Jacket Black             | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAEJ030817   | Women's Long Sleeve Blended Cardigan Charcoal       | 8               | 9          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAC080916   | Android Lifted Men's Short Sleeve Tee Blue          | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALL074616   | Women's Short Sleeve Badge Tee Navy                 | 14              | 15         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0732     | Women's Fleece Hoodie Black                         | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAEC029114   | Women's Short Sleeve Hero Tee Sky Blue              | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADC059515   | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAUC057714   | Women's Performance Golf Polo Blue                  | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPB058213   | Women's Lightweight Microfleece Jacket              | 15              | 16         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAL081113   | Android Men's Pep Rally Short Sleeve Tee Navy       | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALQ036616   | Women's Scoop Neck Tee White                        | 26              | 27         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0688     | Android Youth Short Sleeve T-shirt Pewter           | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0584     | Women's Yoga Pants                                  | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAC035016   | Men's Bayside Graphic Tee                           | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAXJ065730   | Toddler 1/4 Zip Fleece Pewter                       | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0284     | Women's  Short Sleeve Hero Tee   Black              | 22              | 23         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADC059317   | Men's Airflow 1/4 Zip Pullover Lapis                | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAL059013   | Men's Short Sleeve Performance Badge Tee Navy       | 18              | 19         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAJ057018   | Men's Short Sleeve Performance Badge Tee Charcoal   | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYQ069026   | Youth Short Sleeve Tee White                        | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0348     | Android BTTF Cosmos Graphic Tee                     | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAWT061749   | Android Onesie Gold                                 | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAEJ028216   | Android Women's Short Sleeve Badge Tee Dark Heather | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAJ080817   | Android Men's Engineer Short Sleeve Tee Charcoal    | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPB058216   | Women's Lightweight Microfleece Jacket              | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPB057812   | Women's Performance Full Zip Jacket Black           | 8               | 9          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0662     | Android Toddler Short Sleeve T-shirt Pewter         | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAJ057016   | Men's Short Sleeve Performance Badge Tee Charcoal   | 29              | 30         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAUC057713   | Women's Performance Golf Polo Blue                  | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAUC057717   | Women's Performance Golf Polo Blue                  | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGARJ058413   | Women's Yoga Pants                                  | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPB058615   | Women's Yoga Jacket Black                           | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAXR066130   | Toddler Short Sleeve Tee Red                        | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAB058916   | Men's Short Sleeve Performance Badge Tee Black      | 10              | 11         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAP081217   | Android Men's Take Charge Short Sleeve Tee Purple   | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADC059314   | Men's Airflow 1/4 Zip Pullover Lapis                | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYQ069024   | Youth Short Sleeve Tee White                        | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYB068025   | Youth Baseball Raglan Heather/Black                 | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYQ069025   | Youth Short Sleeve Tee White                        | 10              | 11         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPJ058016   | Women's 1/4 Zip Jacket Charcoal                     | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAWR061850   | Infant Short Sleeve Tee Red                         | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGATB060413   | Women's Quilted Insulated Vest Black                | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAJ032413   | Android Men's Short Sleeve Tri-blend Hero Tee Grey  | 11              | 12         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAB058913   | Men's Short Sleeve Performance Badge Tee Black      | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0566     | Men's Weatherblock Shell Jacket Black               | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAEJ029014   | Women's Short Sleeve Hero Tee Charcoal              | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADC059517   | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0746     | Women's Short Sleeve Badge Tee Navy                 | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAXJ066255   | Android Toddler Short Sleeve T-shirt Pewter         | 16              | 17         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0683     | Youth Short Sleeve T-shirt Royal Blue               | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0679     | Youth Girl Tee Green                                | 14              | 15         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAXR065129   | Toddler Short Sleeve Tee Red                        | 11              | 12         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALJ060116   | Women's Short Sleeve Performance Tee Pewter         | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYQ068924   | Youth Girl Hoodie                                   | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAEJ029914   | Women's V-Neck Tee Charcoal                         | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0614     | Onesie Heather                                      | 25              | 26         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAXQ067155   | Toddler Short Sleeve Tee White                      | 8               | 9          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAXXX0810     | Android Men's Outerstellar Short Sleeve Tee Black   | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAXR066155   | Toddler Short Sleeve Tee Red                        | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0321     | Men's Short Sleeve Hero Tee Light Blue              | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAPJ058014   | Women's 1/4 Zip Jacket Charcoal                     | 17              | 18         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAEJ028214   | Android Women's Short Sleeve Badge Tee Dark Heather | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAER029715   | Women's Short Sleeve Hero Tee Red Heather           | 13              | 14         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADJ056818   | Men's Watershed Full Zip Hoodie Grey                | 6               | 7          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAAQ031717   | Men's Short Sleeve Hero Tee  White                  | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAB034815   | Android BTTF Cosmos Graphic Tee                     | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEAAAH083313   | Android Men's Paradise Short Sleeve Tee Olive       | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALP034316   | Women's Vintage Hero Tee Lavender                   | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAXR066028   | Toddler Sports T-shirt Red                          | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADJ059718   | Men's Quilted Insulated Vest Battleship Grey        | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAEJ029616   | Women's Short Sleeve Tri-blend Badge Tee Grey       | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADJ057115   | Men's Performance 1/4 Zip Pullover Heather/Black    | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAER035516   | Men's Vintage Tank                                  | 5               | 6          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0319     | Android Men's Short Sleeve Hero Tee Heather         | 18              | 19         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGXXX0845     | BLM Sweatshirt                                      | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAEJ029017   | Women's Short Sleeve Hero Tee Charcoal              | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGATJ060513   | Women's Quilted Insulated Vest White                | 17              | 18         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGALJ073315   | Women's Short Sleeve Hero Tee Heather               | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAEJ029613   | Women's Short Sleeve Tri-blend Badge Tee Grey       | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGADB056717   | Men's Softshell Jacket Black/Grey                   | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGATH060715   | Women's Convertible Vest-Jacket Sea Foam Green      | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAEJ031316   | Tri-blend Hoodie Grey                               | 10              | 11         | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEWALJ082713   | Women's Short Sleeve Tee                            | 3               | 4          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEYAEJ029617   | Women's Short Sleeve Tri-blend Badge Tee Grey       | 2               | 3          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAAX0281     | Women's Short Sleeve Badge Tee Grey                 | 7               | 8          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAEJ028917   | Women's Short Sleeve Hero Dark Grey                 | 4               | 5          | 1                | GREEN: OK for fulfill      | 17                      |
| GGOEGAYH067925   | Youth Girl Tee Green                                | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ028114   | Women's Short Sleeve Badge Tee Grey                 | 19              | 21         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAFB073116   | Men's Fleece Hoodie Black                           | 20              | 22         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEB028313   | Android Women's Short Sleeve Hero Tee Black         | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAQ057215   | Men's Colorblock Tee White/Heather                  | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAYJ068826   | Android Youth Short Sleeve T-shirt Pewter           | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAYQ068956   | Youth Girl Hoodie                                   | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0611     | Onesie Red                                          | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWH061349   | Onesie Green                                        | 19              | 21         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGXXX0806     | Men's Bike Short Sleeve Tee Charcoal                | 10              | 12         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAJ057017   | Men's Short Sleeve Performance Badge Tee Charcoal   | 7               | 9          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEH035215   | Android Men's Vintage Henley                        | 25              | 27         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGALJ034417   | Women's Vintage Hero Tee Platinum                   | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAXN066328   | Android Toddler Short Sleeve T-shirt Pink           | 20              | 22         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGALP036317   | Women's Long Sleeve Tee Lavender                    | 7               | 9          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWH061350   | Onesie Green                                        | 7               | 9          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0573     | Men's Lightweight Microfleece Jacket Black          | 18              | 20         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAL010617   | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGADJ057116   | Men's Performance 1/4 Zip Pullover Heather/Black    | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0282     | Android Women's Short Sleeve Badge Tee Dark Heather | 17              | 19         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ028913   | Women's Short Sleeve Hero Dark Grey                 | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAPB058214   | Women's Lightweight Microfleece Jacket              | 11              | 13         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAXC065230   | Toddler Short Sleeve T-shirt Royal Blue             | 10              | 12         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEJ030916   | Android Women's Long Sleeve Blended Cardigan Grey   | 8               | 10         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB081013   | Android Men's Outerstellar Short Sleeve Tee Black   | 10              | 12         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0690     | Youth Short Sleeve Tee White                        | 29              | 31         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB034914   | Android BTTF Moonshot Graphic Tee                   | 10              | 12         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB034813   | Android BTTF Cosmos Graphic Tee                     | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGALL074614   | Women's Short Sleeve Badge Tee Navy                 | 29              | 31         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAHPB076114   | Android Stretch Fit Hat                             | 6               | 8          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGHPL003214   | Stretch Fit Hat M/L Navy                            | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAPB058617   | Women's Yoga Jacket Black                           | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAEB028415   | Women's  Short Sleeve Hero Tee   Black              | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGADB059216   | Men's Airflow 1/4 Zip Pullover Black                | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWH061353   | Onesie Green                                        | 27              | 29         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ028914   | Women's Short Sleeve Hero Dark Grey                 | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0591     | Men's Short Sleeve Performance Badge Tee Pewter     | 27              | 29         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAXL065928   | Toddler Raglan Shirt Blue Heather/Navy              | 10              | 12         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAAQ031715   | Men's Short Sleeve Hero Tee White                   | 37              | 39         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGADB057313   | Men's Lightweight Microfleece Jacket Black          | 9               | 11         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGATB060616   | Women's Convertible Vest-Jacket Black               | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAQ010416   | Men's 100% Cotton Short Sleeve Hero Tee White       | 9               | 11         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGADC059316   | Men's Airflow 1/4 Zip Pullover Lapis                | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGALJ036415   | Women's Tee Grey                                    | 21              | 23         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAEJ029516   | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 20              | 22         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0327     | Men's Long & Lean Tee Grey                          | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAAB031816   | Men's Short Sleeve Hero Tee Black                   | 18              | 20         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAYR068226   | Youth Short Sleeve Tee Red                          | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEYAEB035616   | Men's Vintage Tank                                  | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWR061050   | Onesie Red/Graphite                                 | 4               | 6          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAYH067926   | Youth Girl Tee Green                                | 20              | 22         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB034916   | Android BTTF Moonshot Graphic Tee                   | 11              | 13         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWR061150   | Onesie Red                                          | 20              | 22         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0331     | Men's Long Sleeve Raglan Ocean Blue                 | 8               | 10         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAXL065930   | Toddler Raglan Shirt Blue Heather/Navy              | 18              | 20         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAP081216   | Android Men's Take Charge Short Sleeve Tee Purple   | 14              | 16         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAER029717   | Women's Short Sleeve Hero Tee Red Heather           | 5               | 7          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ035314   | Vintage Henley Grey/Black                           | 19              | 21         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAYR068256   | Youth Short Sleeve Tee Red                          | 8               | 10         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEJ035713   | Android Men's Vintage Tank                          | 13              | 15         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ029913   | Women's V-Neck Tee Charcoal                         | 6               | 8          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB034816   | Android BTTF Cosmos Graphic Tee                     | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAJ059116   | Men's Short Sleeve Performance Badge Tee Pewter     | 13              | 15         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEH035217   | Android Men's Vintage Henley                        | 8               | 10         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEJ030915   | Android Women's Long Sleeve Blended Cardigan Grey   | 6               | 8          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAXR066030   | Toddler Sports T-shirt Red                          | 7               | 9          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAB081015   | Android Men's Outerstellar Short Sleeve Tee Black   | 11              | 13         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAHPB076113   | Android Stretch Fit Hat                             | 8               | 10         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0344     | Women's Vintage Hero Tee Platinum                   | 32              | 34         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGALJ072916   | Women's Performance Hero Tee Gunmetal               | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAFJ036213   | Men's Pullover Hoodie Grey                          | 12              | 14         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAEJ028115   | Women's Short Sleeve Badge Tee Grey                 | 11              | 13         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAWR061053   | Onesie Red/Graphite                                 | 12              | 14         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAX0588     | Women's Shell Jacket Blue/Black                     | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAAH083317   | Android Men's Paradise Short Sleeve Tee Olive       | 9               | 11         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGATH060716   | Women's Convertible Vest-Jacket Sea Foam Green      | 9               | 11         | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGATJ060516   | Women's Quilted Insulated Vest White                | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAPB058614   | Women's Yoga Jacket Black                           | 6               | 8          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAYJ068856   | Android Youth Short Sleeve T-shirt Pewter           | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGAAC032116   | Men's Short Sleeve Hero Tee Light Blue              | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEAAEJ028215   | Android Women's Short Sleeve Badge Tee Dark Heather | 3               | 5          | 2                | GREEN: OK for fulfill      | 18                      |
| GGOEGATB060214   | Women's 1/4 Zip Performance Pullover Black          | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEYAAJ033013   | Men's Long & Lean Tee Charcoal                      | 10              | 13         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEWFKA083199   | Mood Original Window Decal                          | 63              | 66         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADJ059814   | Men's Convertible Vest-Jacket Pewter                | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGPXR023199   | Collapsible Pet Bowl                                | 12              | 15         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAWQ062248   | Infant Short Sleeve Tee White                       | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALQ036614   | Women's Scoop Neck Tee White                        | 49              | 52         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGATB060216   | Women's 1/4 Zip Performance Pullover Black          | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADB057318   | Men's Lightweight Microfleece Jacket Black          | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAWC061950   | Infant Short Sleeve Tee Royal Blue                  | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAXJ065755   | Toddler 1/4 Zip Fleece Pewter                       | 26              | 29         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALJ036413   | Women's Tee Grey                                    | 9               | 12         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAFB035917   | Android Men's  Zip Hoodie                           | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADJ057114   | Men's Performance 1/4 Zip Pullover Heather/Black    | 16              | 19         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAEB084515   | BLM Sweatshirt                                      | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0628     | Infant Zip Hood Royal Blue                          | 26              | 29         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGBRP037599   | Yoga Mat Purple                                     | 10              | 13         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEYAAB031815   | Men's Short Sleeve Hero Tee Black                   | 36              | 39         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAYC068326   | Youth Short Sleeve T-shirt Royal Blue               | 11              | 14         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALP036314   | Women's Long Sleeve Tee Lavender                    | 13              | 16         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAEJ030814   | Women's Long Sleeve Blended Cardigan Charcoal       | 34              | 37         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAB058917   | Men's Short Sleeve Performance Badge Tee Black      | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAJ080614   | Men's Bike Short Sleeve Tee Charcoal                | 50              | 53         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0607     | Women's Convertible Vest-Jacket Sea Foam Green      | 22              | 25         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAAB034817   | Android BTTF Cosmos Graphic Tee                     | 7               | 10         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAYH068425   | Youth Short Sleeve T-shirt Green                    | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAYQ068925   | Youth Girl Hoodie                                   | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAPJ058012   | Women's 1/4 Zip Jacket Charcoal                     | 9               | 12         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAXJ065728   | Toddler 1/4 Zip Fleece Pewter                       | 27              | 30         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALJ057914   | Women's Short Sleeve Performance Tee Charcoal       | 11              | 14         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADB056917   | Men's Performance Full Zip Jacket Black             | 7               | 10         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAB010513   | Men's 100% Cotton Short Sleeve Hero Tee Black       | 32              | 35         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOENEBQ081599   | Cam Outdoor Security Camera - CA                    | 37              | 40         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEYAAJ032513   | Men's Short Sleeve Hero Tee Charcoal                | 20              | 23         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAAC080917   | Android Lifted Men's Short Sleeve Tee Blue          | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAC035017   | Men's Bayside Graphic Tee                           | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0689     | Youth Girl Hoodie                                   | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0328     | Men's Long & Lean Tee Charcoal                      | 8               | 11         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAXL065955   | Toddler Raglan Shirt Blue Heather/Navy              | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAWN062650   | Android Infant Short Sleeve Tee Pink                | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADB059217   | Men's Airflow 1/4 Zip Pullover Black                | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAJ073415   | Men's Short Sleeve Hero Tee Heather                 | 10              | 13         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAYC068724   | Android Youth Short Sleeve T-shirt Aqua             | 25              | 28         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0580     | Women's 1/4 Zip Jacket Charcoal                     | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADB056616   | Men's Weatherblock Shell Jacket Black               | 13              | 16         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAWJ062549   | Android Infant Short Sleeve Tee Pewter              | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAXXX0809     | Android Lifted Men's Short Sleeve Tee Blue          | 11              | 14         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAAJ032913   | Android Men's Long & Lean Badge Tee Charcoal        | 24              | 27         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALP036315   | Women's Long Sleeve Tee Lavender                    | 15              | 18         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEAAAH083316   | Android Men's Paradise Short Sleeve Tee Olive       | 8               | 11         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAPB057817   | Women's Performance Full Zip Jacket Black           | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADJ059714   | Men's Quilted Insulated Vest Battleship Grey        | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGATC060315   | Women's 1/4 Zip Performance Pullover Two-Tone Blue  | 5               | 8          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALB036516   | Women's Scoop Neck Tee Black                        | 20              | 23         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEYAXR066129   | Toddler Short Sleeve Tee Red                        | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGALQ036617   | Women's Scoop Neck Tee White                        | 7               | 10         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGADC059513   | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAEC033117   | Men's Long Sleeve Raglan Ocean Blue                 | 6               | 9          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAER035513   | Men's Vintage Tank                                  | 4               | 7          | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAL010615   | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 63              | 66         | 3                | GREEN: OK for fulfill      | 19                      |
| GGOEGAAX0283     | Android Women's Short Sleeve Hero Tee Black         | 7               | 11         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGADB057316   | Men's Lightweight Microfleece Jacket Black          | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAEJ035716   | Android Men's Vintage Tank                          | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAHPB080210   | Android 5-Panel Low Cap                             | 7               | 11         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAXQ067128   | Toddler Short Sleeve Tee White                      | 8               | 12         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAC035014   | Men's Bayside Graphic Tee                           | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGALP034317   | Women's Vintage Hero Tee Lavender                   | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAFB035913   | Android Men's  Zip Hoodie                           | 8               | 12         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGALJ036416   | Women's Tee Grey                                    | 7               | 11         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGALJ073314   | Women's Short Sleeve Hero Tee Heather               | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAAJ032417   | Android Men's Short Sleeve Tri-blend Hero Tee Grey  | 8               | 12         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGEHQ072499   | 2200mAh Micro Charger                               | 34              | 38         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAEJ033417   | Android Men's Long Sleeve Badge Crew Tee Heather    | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAJ032617   | Men's Short Sleeve Badge Tee Charcoal               | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAYR068224   | Youth Short Sleeve Tee Red                          | 9               | 13         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAYQ069056   | Youth Short Sleeve Tee White                        | 10              | 14         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAPB058612   | Women's Yoga Jacket Black                           | 17              | 21         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEYAQB073216   | Women's Fleece Hoodie Black                         | 9               | 13         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGADB059616   | Men's Quilted Insulated Vest Black                  | 13              | 17         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAWC061948   | Infant Short Sleeve Tee Royal Blue                  | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAXL065929   | Toddler Raglan Shirt Blue Heather/Navy              | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAJ073016   | Men's Performance Hero Tee Gunmetal                 | 6               | 10         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAQ057213   | Men's Colorblock Tee White/Heather                  | 6               | 10         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0326     | Men's Short Sleeve Badge Tee Charcoal               | 8               | 12         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAXJ065729   | Toddler 1/4 Zip Fleece Pewter                       | 29              | 33         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0604     | Women's Quilted Insulated Vest Black                | 6               | 10         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGESB015199   | Flashlight                                          | 9               | 13         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAAC080913   | Android Lifted Men's Short Sleeve Tee Blue          | 7               | 11         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEAAAJ080816   | Android Men's Engineer Short Sleeve Tee Charcoal    | 9               | 13         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0681     | Youth Sports Tee Navy                               | 29              | 33         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0595     | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 5               | 9          | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0664     | Android Toddler Short Sleeve T-shirt Aqua           | 12              | 16         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAQ033918   | Men's Vintage Badge Tee White                       | 12              | 16         | 4                | GREEN: OK for fulfill      | 20                      |
| GGOEGAAX0330     | Men's Long & Lean Tee Charcoal                      | 38              | 43         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEWAEA083899   | Dress Socks                                         | 15              | 20         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAPB057816   | Women's Performance Full Zip Jacket Black           | 6               | 11         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAXR066029   | Toddler Sports T-shirt Red                          | 8               | 13         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEAAAJ031916   | Android Men's Short Sleeve Hero Tee Heather         | 79              | 84         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0361     | Android Women's Fleece Hoodie                       | 16              | 21         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEAAWN062649   | Android Infant Short Sleeve Tee Pink                | 13              | 18         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEYAYR068656   | Youth Short Sleeve Tee Red                          | 10              | 15         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAEQ027916   | Women's Short Sleeve Hero Tee White                 | 14              | 19         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0297     | Women's Short Sleeve Hero Tee Red Heather           | 11              | 16         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAEC029115   | Women's Short Sleeve Hero Tee Sky Blue              | 11              | 16         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGATH060713   | Women's Convertible Vest-Jacket Sea Foam Green      | 27              | 32         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0359     | Android Men's  Zip Hoodie                           | 12              | 17         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAJ073416   | Men's Short Sleeve Hero Tee Heather                 | 18              | 23         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEAAAP081213   | Android Men's Take Charge Short Sleeve Tee Purple   | 16              | 21         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGALJ057913   | Women's Short Sleeve Performance Tee Charcoal       | 6               | 11         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEAAAL081116   | Android Men's Pep Rally Short Sleeve Tee Navy       | 14              | 19         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOBJGOWUSG69402 | USB wired soundbar - in store only                  | 10              | 15         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0603     | Women's 1/4 Zip Performance Pullover Two-Tone Blue  | 10              | 15         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGALJ057915   | Women's Short Sleeve Performance Tee Charcoal       | 6               | 11         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAJ080613   | Men's Bike Short Sleeve Tee Charcoal                | 24              | 29         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGALQ034216   | Women's Vintage Hero Tee White                      | 12              | 17         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEYAAB031814   | Men's Short Sleeve Hero Tee Black                   | 35              | 40         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAXQ067129   | Toddler Short Sleeve Tee White                      | 7               | 12         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0306     | Women's Recycled Fabric Tee                         | 11              | 16         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0661     | Toddler Short Sleeve Tee Red                        | 12              | 17         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAQB036016   | Women's Fleece Hoodie                               | 6               | 11         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAFJ036216   | Men's Pullover Hoodie Grey                          | 12              | 17         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAEJ030816   | Women's Long Sleeve Blended Cardigan Charcoal       | 9               | 14         | 5                | GREEN: OK for fulfill      | 21                      |
| GGOEGAAX0590     | Men's Short Sleeve Performance Badge Tee Navy       | 15              | 21         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAAX0605     | Women's Quilted Insulated Vest White                | 10              | 16         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAFB035817   | Men's  Zip Hoodie                                   | 9               | 15         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAAL010616   | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 44              | 50         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGALJ036417   | Women's Tee Grey                                    | 7               | 13         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAEJ031315   | Tri-blend Hoodie Grey                               | 28              | 34         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGALQ034217   | Women's Vintage Hero Tee White                      | 13              | 19         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGATB060617   | Women's Convertible Vest-Jacket Black               | 9               | 15         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAXQ067130   | Toddler Short Sleeve Tee White                      | 12              | 18         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAAX0589     | Men's Short Sleeve Performance Badge Tee Black      | 9               | 15         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAER029716   | Women's Short Sleeve Hero Tee Red Heather           | 7               | 13         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEAAQB036116   | Android Women's Fleece Hoodie                       | 12              | 18         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAWR061153   | Onesie Red                                          | 14              | 20         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEAAAB034915   | Android BTTF Moonshot Graphic Tee                   | 10              | 16         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEAAFB035915   | Android Men's  Zip Hoodie                           | 8               | 14         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAXC065229   | Toddler Short Sleeve T-shirt Royal Blue             | 16              | 22         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAFJ036215   | Men's Pullover Hoodie Grey                          | 21              | 27         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEYAEB028414   | Women's  Short Sleeve Hero Tee   Black              | 31              | 37         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAXC065228   | Toddler Short Sleeve T-shirt Royal Blue             | 14              | 20         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEAAAH083315   | Android Men's Paradise Short Sleeve Tee Olive       | 12              | 18         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGADB059613   | Men's Quilted Insulated Vest Black                  | 12              | 18         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGAAX0596     | Men's Quilted Insulated Vest Black                  | 26              | 32         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGALJ034416   | Women's Vintage Hero Tee Platinum                   | 7               | 13         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGATB060614   | Women's Convertible Vest-Jacket Black               | 31              | 37         | 6                | GREEN: OK for fulfill      | 22                      |
| GGOEGADB059614   | Men's Quilted Insulated Vest Black                  | 20              | 27         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEAXXX0812     | Android Men's Take Charge Short Sleeve Tee Purple   | 12              | 19         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGALB036515   | Women's Scoop Neck Tee Black                        | 55              | 62         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGALQ036615   | Women's Scoop Neck Tee White                        | 43              | 50         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGATC060313   | Women's 1/4 Zip Performance Pullover Two-Tone Blue  | 21              | 28         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEAAXN066355   | Android Toddler Short Sleeve T-shirt Pink           | 21              | 28         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0304     | Women's 3/4 Sleeve Baseball Raglan Heather/Black    | 19              | 26         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0618     | Infant Short Sleeve Tee Red                         | 8               | 15         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0578     | Women's Performance Full Zip Jacket Black           | 10              | 17         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEAAAB034917   | Android BTTF Moonshot Graphic Tee                   | 13              | 20         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGADC059514   | Men's Microfiber 1/4 Zip Pullover Blue/Indigo       | 9               | 16         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEYAEJ029514   | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 81              | 88         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0625     | Android Infant Short Sleeve Tee Pewter              | 16              | 23         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0663     | Android Toddler Short Sleeve T-shirt Pink           | 12              | 19         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAWR061848   | Infant Short Sleeve Tee Red                         | 11              | 18         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAR010715   | Men's 100% Cotton Short Sleeve Hero Tee Red         | 139             | 146        | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEYAAJ032516   | Men's Short Sleeve Hero Tee Charcoal                | 11              | 18         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEYAFB073117   | Men's Fleece Hoodie Black                           | 13              | 20         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAEJ028113   | Women's Short Sleeve Badge Tee Grey                 | 19              | 26         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAWR061149   | Onesie Red                                          | 24              | 31         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAYL068124   | Youth Sports Tee Navy                               | 22              | 29         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAJ032317   | Men's Short Sleeve Hero Tee Charcoal                | 18              | 25         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAAX0593     | Men's Airflow 1/4 Zip Pullover Lapis                | 16              | 23         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGALJ036414   | Women's Tee Grey                                    | 16              | 23         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGAYH067956   | Youth Girl Tee Green                                | 11              | 18         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEYAEA035116   | Men's Vintage Henley                                | 19              | 26         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEGADC059315   | Men's Airflow 1/4 Zip Pullover Lapis                | 24              | 31         | 7                | GREEN: OK for fulfill      | 23                      |
| GGOEAAEJ035715   | Android Men's Vintage Tank                          | 15              | 23         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAX0626     | Android Infant Short Sleeve Tee Pink                | 13              | 21         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAJ032713   | Men's Long & Lean Tee Grey                          | 13              | 21         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAC032113   | Men's Short Sleeve Hero Tee Light Blue              | 9               | 17         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEYAAJ033017   | Men's Long & Lean Tee Charcoal                      | 12              | 20         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAX0362     | Men's Pullover Hoodie Grey                          | 25              | 33         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAX0365     | Women's Scoop Neck Tee Black                        | 65              | 73         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAX0579     | Women's Short Sleeve Performance Tee Charcoal       | 10              | 18         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGADJ057113   | Men's Performance 1/4 Zip Pullover Heather/Black    | 10              | 18         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEWCKQ085457   | Pack of 9 Decal Set                                 | 31              | 39         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAEA030413   | Womens 3/4 Sleeve Baseball Raglan Heather/Black     | 23              | 31         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEYAYR068625   | Youth Short Sleeve Tee Red                          | 18              | 26         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEAAXC066455   | Android Toddler Short Sleeve T-shirt Aqua           | 18              | 26         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGALJ034414   | Women's Vintage Hero Tee Platinum                   | 34              | 42         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEYAAB031817   | Men's Short Sleeve Hero Tee Black                   | 13              | 21         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEYAWJ061454   | Onesie Heather                                      | 13              | 21         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGALQ034215   | Women's Vintage Hero Tee White                      | 29              | 37         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAAJ032717   | Men's Long & Lean Tee Grey                          | 14              | 22         | 8                | GREEN: OK for fulfill      | 24                      |
| GGOEGAXC065630   | Toddler Hoodie Royal Blue                           | 18              | 27         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAX0602     | Women's 1/4 Zip Performance Pullover Black          | 12              | 21         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGALP036313   | Women's Long Sleeve Tee Lavender                    | 20              | 29         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAC035013   | Men's Bayside Graphic Tee                           | 16              | 25         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAWC062850   | Infant Zip Hood Royal Blue                          | 21              | 30         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEAAAJ080813   | Android Men's Engineer Short Sleeve Tee Charcoal    | 17              | 26         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEAAEH035213   | Android Men's Vintage Henley                        | 10              | 19         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGADJ059715   | Men's Quilted Insulated Vest Battleship Grey        | 30              | 39         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEWFKA083099   | Mood Happy Window Decal                             | 17              | 26         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEAAXC066430   | Android Toddler Short Sleeve T-shirt Aqua           | 63              | 72         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGACB023699   | Yoga Block                                          | 69              | 78         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAQ010417   | Men's 100% Cotton Short Sleeve Hero Tee White       | 32              | 41         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAEC029113   | Women's Short Sleeve Hero Tee Sky Blue              | 15              | 24         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAWQ062249   | Infant Short Sleeve Tee White                       | 22              | 31         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAX0571     | Men's Performance 1/4 Zip Pullover Heather/Black    | 17              | 26         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGATH060714   | Women's Convertible Vest-Jacket Sea Foam Green      | 10              | 19         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAQ033915   | Men's Vintage Badge Tee White                       | 111             | 120        | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAC035015   | Men's Bayside Graphic Tee                           | 12              | 21         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAEA030414   | Womens 3/4 Sleeve Baseball Raglan Heather/Black     | 23              | 32         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAX0323     | Men's Short Sleeve Hero Tee Charcoal                | 11              | 20         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAAX0290     | Women's Short Sleeve Hero Tee Charcoal              | 45              | 54         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGATB060615   | Women's Convertible Vest-Jacket Black               | 16              | 25         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEGAYB068026   | Youth Baseball Raglan Heather/Black                 | 12              | 21         | 9                | GREEN: OK for fulfill      | 25                      |
| GGOEYAAJ033016   | Men's Long & Lean Tee Charcoal                      | 12              | 22         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGCBC016999   | Pet Feeding Mat                                     | 105             | 115        | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAJ032716   | Men's Long & Lean Tee Grey                          | 16              | 26         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAJ032314   | Men's Short Sleeve Hero Tee Charcoal                | 49              | 59         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAEC033113   | Men's Long Sleeve Raglan Ocean Blue                 | 26              | 36         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEAAAJ080814   | Android Men's Engineer Short Sleeve Tee Charcoal    | 12              | 22         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAX0309     | Android Women's Long Sleeve Blended Cardigan Grey   | 18              | 28         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEAOBH078799   | Android Luggage Tag                                 | 22              | 32         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAEJ035316   | Vintage Henley Grey/Black                           | 13              | 23         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAYB068024   | Youth Baseball Raglan Heather/Black                 | 11              | 21         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAWR061154   | Onesie Red                                          | 27              | 37         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAC032115   | Men's Short Sleeve Hero Tee Light Blue              | 16              | 26         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAEJ028116   | Women's Short Sleeve Badge Tee Grey                 | 11              | 21         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGALJ072914   | Women's Performance Hero Tee Gunmetal               | 15              | 25         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAX0586     | Women's Yoga Jacket Black                           | 23              | 33         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGALB036517   | Women's Scoop Neck Tee Black                        | 13              | 23         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEGAAX0659     | Toddler Raglan Shirt Blue Heather/Navy              | 12              | 22         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEAAXC066428   | Android Toddler Short Sleeve T-shirt Aqua           | 15              | 25         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEYAEJ029013   | Women's Short Sleeve Hero Tee Charcoal              | 11              | 21         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOEAAAJ032917   | Android Men's Long & Lean Badge Tee Charcoal        | 20              | 30         | 10               | GREEN: OK for fulfill      | 26                      |
| GGOENEBQ081699   | Protect Smoke + CO White Battery Alarm - CA         | 14              | 25         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAER029714   | Women's Short Sleeve Hero Tee Red Heather           | 29              | 40         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEAAYJ068824   | Android Youth Short Sleeve T-shirt Pewter           | 27              | 38         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAER029713   | Women's Short Sleeve Hero Tee Red Heather           | 14              | 25         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAAX0685     | Youth Short Sleeve T-shirt Yellow                   | 27              | 38         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGATB060613   | Women's Convertible Vest-Jacket Black               | 33              | 44         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEAAAL081114   | Android Men's Pep Rally Short Sleeve Tee Navy       | 15              | 26         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGALL074613   | Women's Short Sleeve Badge Tee Navy                 | 22              | 33         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAWR061048   | Onesie Red/Graphite                                 | 20              | 31         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAAX0652     | Toddler Short Sleeve T-shirt Royal Blue             | 19              | 30         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAAX0606     | Women's Convertible Vest-Jacket Black               | 37              | 48         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGEHQ071199   | High Capacity 10,400mAh Charger                     | 73              | 84         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAAX0680     | Youth Baseball Raglan Heather/Black                 | 17              | 28         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEAAAB081014   | Android Men's Outerstellar Short Sleeve Tee Black   | 13              | 24         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAFJ036214   | Men's Pullover Hoodie Grey                          | 13              | 24         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGAWN062750   | Infant Zip Hood Pink                                | 86              | 97         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEGALL074615   | Women's Short Sleeve Badge Tee Navy                 | 14              | 25         | 11               | GREEN: OK for fulfill      | 27                      |
| GGOEAAAH083314   | Android Men's Paradise Short Sleeve Tee Olive       | 18              | 30         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEYAAB031813   | Men's Short Sleeve Hero Tee Black                   | 15              | 27         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGADJ056816   | Men's Watershed Full Zip Hoodie Grey                | 13              | 25         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEYAAQ031716   | Men's Short Sleeve Hero Tee White                   | 20              | 32         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAEJ030815   | Women's Long Sleeve Blended Cardigan Charcoal       | 13              | 25         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAAL010614   | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 62              | 74         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEAAEJ030914   | Android Women's Long Sleeve Blended Cardigan Grey   | 17              | 29         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAAX0610     | Onesie Red/Graphite                                 | 51              | 63         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAAJ059113   | Men's Short Sleeve Performance Badge Tee Pewter     | 13              | 25         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGALP034315   | Women's Vintage Hero Tee Lavender                   | 14              | 26         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAEA030416   | Womens 3/4 Sleeve Baseball Raglan Heather/Black     | 13              | 25         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGBRB079599   | 25L Classic Rucksack                                | 28              | 40         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEYAYR068624   | Youth Short Sleeve Tee Red                          | 26              | 38         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEYAAJ033015   | Men's Long & Lean Tee Charcoal                      | 21              | 33         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGADB056916   | Men's Performance Full Zip Jacket Black             | 15              | 27         | 12               | GREEN: OK for fulfill      | 28                      |
| GGOEGAAR010713   | Men's 100% Cotton Short Sleeve Hero Tee Red         | 21              | 34         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGAWN062748   | Infant Zip Hood Pink                                | 41              | 54         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEAAWN062648   | Android Infant Short Sleeve Tee Pink                | 19              | 32         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEAOCB077499   | Android RFID Journal                                | 73              | 86         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEAAYC068756   | Android Youth Short Sleeve T-shirt Aqua             | 64              | 77         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEYAAJ033014   | Men's Long & Lean Tee Charcoal                      | 29              | 42         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGAAX0657     | Toddler 1/4 Zip Fleece Pewter                       | 31              | 44         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEYAAJ032514   | Men's Short Sleeve Hero Tee Charcoal                | 28              | 41         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGAXR065128   | Toddler Short Sleeve Tee Red                        | 14              | 27         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGADJ059713   | Men's Quilted Insulated Vest Battleship Grey        | 14              | 27         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGAAX0355     | Men's Vintage Tank                                  | 18              | 31         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEAAEJ033416   | Android Men's Long Sleeve Badge Crew Tee Heather    | 15              | 28         | 13               | GREEN: OK for fulfill      | 29                      |
| GGOEGATB060213   | Women's 1/4 Zip Performance Pullover Black          | 21              | 35         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAR010717   | Men's 100% Cotton Short Sleeve Hero Tee Red         | 15              | 29         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAJ073414   | Men's Short Sleeve Hero Tee Heather                 | 15              | 29         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAX0106     | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 15              | 29         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAB010517   | Men's 100% Cotton Short Sleeve Hero Tee Black       | 36              | 50         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEAAWT061753   | Android Onesie Gold                                 | 15              | 29         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAPB058613   | Women's Yoga Jacket Black                           | 20              | 34         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAWQ062250   | Infant Short Sleeve Tee White                       | 16              | 30         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAJ032613   | Men's Short Sleeve Badge Tee Charcoal               | 32              | 46         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAX0364     | Women's Tee Grey                                    | 26              | 40         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAX0620     | Infant Short Sleeve Tee Green                       | 38              | 52         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAXC065629   | Toddler Hoodie Royal Blue                           | 27              | 41         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEAAAJ032916   | Android Men's Long & Lean Badge Tee Charcoal        | 28              | 42         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAAX0296     | Women's Short Sleeve Tri-blend Badge Tee Grey       | 19              | 33         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGALP034313   | Women's Vintage Hero Tee Lavender                   | 17              | 31         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEAAEJ033413   | Android Men's Long Sleeve Badge Crew Tee Heather    | 29              | 43         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAEJ035313   | Vintage Henley Grey/Black                           | 16              | 30         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGAEA030415   | Womens 3/4 Sleeve Baseball Raglan Heather/Black     | 19              | 33         | 14               | GREEN: OK for fulfill      | 30                      |
| GGOEGEHQ072599   | 4400mAh Power Bank                                  | 50              | 65         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAAJ032313   | Men's Short Sleeve Hero Tee Charcoal                | 46              | 61         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEAAWT061748   | Android Onesie Gold                                 | 19              | 34         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAAJ080617   | Men's Bike Short Sleeve Tee Charcoal                | 19              | 34         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAAC032114   | Men's Short Sleeve Hero Tee Light Blue              | 20              | 35         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAAX0687     | Android Youth Short Sleeve T-shirt Aqua             | 24              | 39         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAQB036013   | Women's Fleece Hoodie                               | 58              | 73         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOENEBJ081899   | Learning Thermostat 3rd Gen - CA - Stainless Steel  | 54              | 69         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGALB034117   | Women's Vintage Hero Tee Black                      | 59              | 74         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAYH068456   | Youth Short Sleeve T-shirt Green                    | 17              | 32         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEYAEA035115   | Men's Vintage Henley                                | 38              | 53         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGBRA037499   | Waterproof Backpack                                 | 215             | 230        | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGBRC021399   | Yoga Mat Blue                                       | 45              | 60         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGAAX0682     | Youth Short Sleeve Tee Red                          | 16              | 31         | 15               | GREEN: OK for fulfill      | 31                      |
| GGOEGADJ056814   | Men's Watershed Full Zip Hoodie Grey                | 41              | 57         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEAAXN066329   | Android Toddler Short Sleeve T-shirt Pink           | 54              | 70         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAJ057013   | Men's Short Sleeve Performance Badge Tee Charcoal   | 27              | 43         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGBRJ037399   | Rucksack                                            | 212             | 228        | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEYAAJ032515   | Men's Short Sleeve Hero Tee Charcoal                | 17              | 33         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAPB057813   | Women's Performance Full Zip Jacket Black           | 17              | 33         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAX0619     | Infant Short Sleeve Tee Royal Blue                  | 17              | 33         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAX0350     | Men's Bayside Graphic Tee                           | 17              | 33         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEAHPB076115   | Android Stretch Fit Hat                             | 18              | 34         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGGCX056399   | Gift Card - $250.00                                 | 22              | 38         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAX0320     | Android Men's Short Sleeve Hero Tee White           | 22              | 38         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAL010613   | Men's 100% Cotton Short Sleeve Hero Tee Navy        | 21              | 37         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAEB027816   | Women's Short Sleeve Hero Tee Black                 | 21              | 37         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGAAX0352     | Android Men's Vintage Henley                        | 21              | 37         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGALQ034213   | Women's Vintage Hero Tee White                      | 78              | 94         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEGALP034314   | Women's Vintage Hero Tee Lavender                   | 27              | 43         | 16               | GREEN: OK for fulfill      | 32                      |
| GGOEYAWJ061453   | Onesie Heather                                      | 26              | 43         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAAX0627     | Infant Zip Hood Pink                                | 18              | 35         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGKWR060810   | Bib Red                                             | 27              | 44         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGALJ060114   | Women's Short Sleeve Performance Tee Pewter         | 27              | 44         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGBRD079699   | 25L Classic Rucksack                                | 23              | 40         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEYAEJ029515   | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 94              | 111        | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAYL068126   | Youth Sports Tee Navy                               | 25              | 42         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGADB056614   | Men's Weatherblock Shell Jacket Black               | 27              | 44         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGADB059215   | Men's Airflow 1/4 Zip Pullover Black                | 24              | 41         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAAX0042     | Android Stretch Fit Hat                             | 24              | 41         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAAX0308     | Women's Long Sleeve Blended Cardigan Charcoal       | 20              | 37         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAXC065655   | Toddler Hoodie Royal Blue                           | 30              | 47         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAAX0291     | Women's Short Sleeve Hero Tee Sky Blue              | 27              | 44         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAWH061354   | Onesie Green                                        | 33              | 50         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAAJ032616   | Men's Short Sleeve Badge Tee Charcoal               | 31              | 48         | 17               | GREEN: OK for fulfill      | 33                      |
| GGOEGAYL068125   | Youth Sports Tee Navy                               | 19              | 37         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEYAWJ061448   | Onesie Heather                                      | 19              | 37         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAEJ028016   | Women's Short Sleeve Hero Tee Grey                  | 33              | 51         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEYAWJ061450   | Onesie Heather                                      | 19              | 37         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEAXXX0808     | Android Men's Engineer Short Sleeve Tee Charcoal    | 37              | 55         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEAAXC066429   | Android Toddler Short Sleeve T-shirt Aqua           | 81              | 99         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAYL068156   | Youth Sports Tee Navy                               | 30              | 48         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAAX0656     | Toddler Hoodie Royal Blue                           | 33              | 51         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAAX0686     | Youth Short Sleeve Tee Red                          | 22              | 40         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEAAAL081115   | Android Men's Pep Rally Short Sleeve Tee Navy       | 23              | 41         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAXC065628   | Toddler Hoodie Royal Blue                           | 27              | 45         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGAER035515   | Men's Vintage Tank                                  | 23              | 41         | 18               | GREEN: OK for fulfill      | 34                      |
| GGOEGADJ056813   | Men's Watershed Full Zip Hoodie Grey                | 35              | 54         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGFAQ016699   | Bottle Opener Clip                                  | 362             | 381        | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGADB059214   | Men's Airflow 1/4 Zip Pullover Black                | 22              | 41         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEAAAJ031913   | Android Men's Short Sleeve Hero Tee Heather         | 49              | 68         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEYAEA035114   | Men's Vintage Henley                                | 46              | 65         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGAAX0671     | Toddler Short Sleeve Tee White                      | 21              | 40         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGAAX0295     | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 24              | 43         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEAAEH035214   | Android Men's Vintage Henley                        | 24              | 43         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEAAAJ032915   | Android Men's Long & Lean Badge Tee Charcoal        | 69              | 88         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGAFB035816   | Men's  Zip Hoodie                                   | 22              | 41         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEGAEJ030813   | Women's Long Sleeve Blended Cardigan Charcoal       | 23              | 42         | 19               | GREEN: OK for fulfill      | 35                      |
| GGOEAAAJ031917   | Android Men's Short Sleeve Hero Tee Heather         | 37              | 57         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAX0231     | Collapsible Pet Bowl                                | 39              | 59         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGALQ036613   | Women's Scoop Neck Tee White                        | 39              | 59         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGADB059615   | Men's Quilted Insulated Vest Black                  | 31              | 51         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEYAFB073113   | Men's Fleece Hoodie Black                           | 79              | 99         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAR010716   | Men's 100% Cotton Short Sleeve Hero Tee Red         | 70              | 90         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAX0613     | Onesie Green                                        | 29              | 49         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAX0360     | Women's Fleece Hoodie                               | 58              | 78         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAX0279     | Women's Short Sleeve Hero Tee White                 | 21              | 41         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAFB035813   | Men's  Zip Hoodie                                   | 21              | 41         | 20               | GREEN: OK for fulfill      | 36                      |
| GGOEGAAX0569     | Men's Performance Full Zip Jacket Black             | 33              | 54         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGPJC019099   | 7 Dog Frisbee                                       | 133             | 154        | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGAAX0622     | Infant Short Sleeve Tee White                       | 34              | 55         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGAAX0597     | Men's Quilted Insulated Vest Battleship Grey        | 24              | 45         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGAYT068524   | Youth Short Sleeve T-shirt Yellow                   | 25              | 46         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEAAAP081215   | Android Men's Take Charge Short Sleeve Tee Purple   | 23              | 44         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGAAX0568     | Men's Watershed Full Zip Hoodie Grey                | 54              | 75         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGCBN016899   | Pet Feeding Mat                                     | 48              | 69         | 21               | GREEN: OK for fulfill      | 37                      |
| GGOEGAAX0280     | Women's Short Sleeve Hero Tee Grey                  | 30              | 52         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEYAEA035117   | Men's Vintage Henley                                | 37              | 59         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEAHPJ004213   | Android Stretch Fit Hat S/M                         | 31              | 53         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEGEVR014999   | Water Resistant Bluetooth Speaker                   | 183             | 205        | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEAAEJ035714   | Android Men's Vintage Tank                          | 42              | 64         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEGAAX0617     | Android Onesie Gold                                 | 31              | 53         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEAHPJ004314   | Android Stretch Fit Hat M/L                         | 37              | 59         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEGAEQ027915   | Women's Short Sleeve Hero Tee White                 | 29              | 51         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEYAEB035615   | Men's Vintage Tank                                  | 52              | 74         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEGAEJ031314   | Tri-blend Hoodie Grey                               | 23              | 45         | 22               | GREEN: OK for fulfill      | 38                      |
| GGOEGPXC023299   | Collapsible Pet Bowl                                | 27              | 50         | 23               | GREEN: OK for fulfill      | 39                      |
| GGOENEBQ081799   | Protect Smoke + CO White Wired Alarm - CA           | 26              | 49         | 23               | GREEN: OK for fulfill      | 39                      |
| GGOEGBRB013899   | Laptop Backpack                                     | 155             | 178        | 23               | GREEN: OK for fulfill      | 39                      |
| GGOEGAEJ028014   | Women's Short Sleeve Hero Tee Grey                  | 90              | 114        | 24               | GREEN: OK for fulfill      | 40                      |
| GGOEGAAX0289     | Women's Short Sleeve Hero Dark Grey                 | 26              | 50         | 24               | GREEN: OK for fulfill      | 40                      |
| GGOEGAAX0357     | Android Men's Vintage Tank                          | 33              | 57         | 24               | GREEN: OK for fulfill      | 40                      |
| GGOEGESC014699   | Aluminum Handy Emergency Flashlight                 | 66              | 90         | 24               | GREEN: OK for fulfill      | 40                      |
| GGOEGAAX0213     | Yoga Mat                                            | 29              | 54         | 25               | GREEN: OK for fulfill      | 41                      |
| GGOEGALB036513   | Women's Scoop Neck Tee Black                        | 66              | 91         | 25               | GREEN: OK for fulfill      | 41                      |
| GGOEGAYT068526   | Youth Short Sleeve T-shirt Yellow                   | 26              | 51         | 25               | GREEN: OK for fulfill      | 41                      |
| GGOEGAAX0761     | Android Stretch Fit Hat Black                       | 27              | 52         | 25               | GREEN: OK for fulfill      | 41                      |
| GGOEGAAX0278     | Women's Short Sleeve Hero Tee Black                 | 95              | 121        | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEAAYC068725   | Android Youth Short Sleeve T-shirt Aqua             | 51              | 77         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEYAWJ061449   | Onesie Heather                                      | 45              | 71         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAAX0299     | Women's V-Neck Tee Charcoal                         | 51              | 77         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEAAXN066330   | Android Toddler Short Sleeve T-shirt Pink           | 44              | 70         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAAX0570     | Men's Short Sleeve Performance Badge Tee Charcoal   | 31              | 57         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAAB033817   | Men's Vintage Badge Tee Black                       | 93              | 119        | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAAX0329     | Android Men's Long & Lean Badge Tee Charcoal        | 39              | 65         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAYT068525   | Youth Short Sleeve T-shirt Yellow                   | 33              | 59         | 26               | GREEN: OK for fulfill      | 42                      |
| GGOEGAAJ032715   | Men's Long & Lean Tee Grey                          | 73              | 100        | 27               | GREEN: OK for fulfill      | 43                      |
| GGOEGAWC062848   | Infant Zip Hood Royal Blue                          | 35              | 62         | 27               | GREEN: OK for fulfill      | 43                      |
| GGOEGAEC033114   | Men's Long Sleeve Raglan Ocean Blue                 | 44              | 71         | 27               | GREEN: OK for fulfill      | 43                      |
| GGOEGAEQ027913   | Women's Short Sleeve Hero Tee White                 | 38              | 65         | 27               | GREEN: OK for fulfill      | 43                      |
| GGOEAAEJ030913   | Android Women's Long Sleeve Blended Cardigan Grey   | 30              | 58         | 28               | GREEN: OK for fulfill      | 44                      |
| GGOEGAAJ080616   | Men's Bike Short Sleeve Tee Charcoal                | 41              | 70         | 29               | GREEN: OK for fulfill      | 45                      |
| GGOEGADB056915   | Men's Performance Full Zip Jacket Black             | 41              | 70         | 29               | GREEN: OK for fulfill      | 45                      |
| GGOEGALJ034415   | Women's Vintage Hero Tee Platinum                   | 36              | 65         | 29               | GREEN: OK for fulfill      | 45                      |
| GGOEGFQB013799   | Compact Selfie Stick                                | 268             | 297        | 29               | GREEN: OK for fulfill      | 45                      |
| GGOEGEVB070899   | Bongo Cupholder Bluetooth Speaker                   | 85              | 115        | 30               | GREEN: OK for fulfill      | 46                      |
| GGOEAAAP081214   | Android Men's Take Charge Short Sleeve Tee Purple   | 38              | 68         | 30               | GREEN: OK for fulfill      | 46                      |
| GGOEGAEJ031313   | Tri-blend Hoodie Grey                               | 32              | 62         | 30               | GREEN: OK for fulfill      | 46                      |
| GGOEGAAX0651     | Toddler Short Sleeve Tee Red                        | 41              | 71         | 30               | GREEN: OK for fulfill      | 46                      |
| GGOEGAAX0366     | Women's Scoop Neck Tee White                        | 51              | 81         | 30               | GREEN: OK for fulfill      | 46                      |
| GGOEGEVB070599   | G Noise-reducing Bluetooth Headphones               | 101             | 132        | 31               | GREEN: OK for fulfill      | 47                      |
| GGOEGBPB082099   | UpCycled Handlebar Bag                              | 76              | 107        | 31               | GREEN: OK for fulfill      | 47                      |
| GGOEGAEC033116   | Men's Long Sleeve Raglan Ocean Blue                 | 39              | 70         | 31               | GREEN: OK for fulfill      | 47                      |
| GGOEGEVB071799   | Pocket Bluetooth Speaker                            | 214             | 246        | 32               | GREEN: OK for fulfill      | 48                      |
| GGOEGAAJ032714   | Men's Long & Lean Tee Grey                          | 49              | 81         | 32               | GREEN: OK for fulfill      | 48                      |
| GGOEGAAX0343     | Women's Vintage Hero Tee Lavender                   | 33              | 65         | 32               | GREEN: OK for fulfill      | 48                      |
| GGOEGAAX0107     | Men's 100% Cotton Short Sleeve Hero Tee Red         | 40              | 72         | 32               | GREEN: OK for fulfill      | 48                      |
| GGOEGGCX056199   | Gift Card- $100.00                                  | 45              | 77         | 32               | GREEN: OK for fulfill      | 48                      |
| GGOEGAAJ032813   | Men's Long & Lean Tee Charcoal                      | 52              | 85         | 33               | GREEN: OK for fulfill      | 49                      |
| GGOEGAAQ033913   | Men's Vintage Badge Tee White                       | 41              | 74         | 33               | GREEN: OK for fulfill      | 49                      |
| GGOEYAEJ029513   | Women's Short Sleeve Tri-blend Badge Tee Charcoal   | 56              | 89         | 33               | GREEN: OK for fulfill      | 49                      |
| GGOEGAAQ010413   | Men's 100% Cotton Short Sleeve Hero Tee White       | 133             | 167        | 34               | GREEN: OK for fulfill      | 50                      |
| GGOEGAAH034017   | Men's Vintage Badge Tee Sage                        | 40              | 74         | 34               | GREEN: OK for fulfill      | 50                      |
| GGOEGCBB074199   | Car Clip Phone Holder                               | 171             | 205        | 34               | GREEN: OK for fulfill      | 50                      |
| GGOENEBB081499   | Cam Indoor Security Camera - CA                     | 38              | 72         | 34               | GREEN: OK for fulfill      | 50                      |
| GGOEGALB034114   | Women's Vintage Hero Tee Black                      | 121             | 155        | 34               | GREEN: OK for fulfill      | 50                      |
| GGOEGALQ034214   | Women's Vintage Hero Tee White                      | 90              | 125        | 35               | GREEN: OK for fulfill      | 51                      |
| GGOEGAAJ032614   | Men's Short Sleeve Badge Tee Charcoal               | 62              | 97         | 35               | GREEN: OK for fulfill      | 51                      |
| GGOEGAAQ033917   | Men's Vintage Badge Tee White                       | 40              | 76         | 36               | GREEN: OK for fulfill      | 52                      |
| GGOEGAAX0317     | Men's Short Sleeve Hero Tee White                   | 50              | 86         | 36               | GREEN: OK for fulfill      | 52                      |
| GGOEADWQ015699   | Android Rise 14 oz Mug                              | 306             | 342        | 36               | GREEN: OK for fulfill      | 52                      |
| GGOEGBPB081999   | UpCycled Bike Saddle Bag                            | 42              | 79         | 37               | GREEN: OK for fulfill      | 53                      |
| GGOEGAXJ065555   | Toddler Short Sleeve T-shirt Grey                   | 56              | 93         | 37               | GREEN: OK for fulfill      | 53                      |
| GGOEGAEB027815   | Women's Short Sleeve Hero Tee Black                 | 163             | 200        | 37               | GREEN: OK for fulfill      | 53                      |
| GGOEGALB034113   | Women's Vintage Hero Tee Black                      | 116             | 153        | 37               | GREEN: OK for fulfill      | 53                      |
| GGOEGAAX0168     | Pet Feeding Mat                                     | 45              | 83         | 38               | GREEN: OK for fulfill      | 54                      |
| GGOEGAAX0353     | Vintage Henley Grey/Black                           | 57              | 95         | 38               | GREEN: OK for fulfill      | 54                      |
| GGOEGALB036514   | Women's Scoop Neck Tee Black                        | 116             | 154        | 38               | GREEN: OK for fulfill      | 54                      |
| GGOEGAAJ059114   | Men's Short Sleeve Performance Badge Tee Pewter     | 69              | 107        | 38               | GREEN: OK for fulfill      | 54                      |
| GGOEA0CH077599   | Android Hard Cover Journal                          | 63              | 102        | 39               | GREEN: OK for fulfill      | 55                      |
| GGOEGAWC062849   | Infant Zip Hood Royal Blue                          | 69              | 108        | 39               | GREEN: OK for fulfill      | 55                      |
| GGOEGAAJ032316   | Men's Short Sleeve Hero Tee Charcoal                | 59              | 98         | 39               | GREEN: OK for fulfill      | 55                      |
| GGOEGHPJ080110   | 5-Panel Cap                                         | 47              | 87         | 40               | GREEN: OK for fulfill      | 56                      |
| GGOEAAEJ033415   | Android Men's Long Sleeve Badge Crew Tee Heather    | 51              | 92         | 41               | GREEN: OK for fulfill      | 57                      |
| GGOEGCBC074299   | Device Stand                                        | 154             | 195        | 41               | GREEN: OK for fulfill      | 57                      |
| GGOEYAEA035113   | Men's Vintage Henley                                | 45              | 87         | 42               | GREEN: OK for fulfill      | 58                      |
| GGOEGAAX0313     | Tri-blend Hoodie Grey                               | 59              | 101        | 42               | GREEN: OK for fulfill      | 58                      |
| GGOEGAAJ032615   | Men's Short Sleeve Badge Tee Charcoal               | 63              | 105        | 42               | GREEN: OK for fulfill      | 58                      |
| GGOEGALB034116   | Women's Vintage Hero Tee Black                      | 79              | 121        | 42               | GREEN: OK for fulfill      | 58                      |
| GGOEGGCX056299   | Gift Card - $25.00                                  | 65              | 107        | 42               | GREEN: OK for fulfill      | 58                      |
| GGOEGAAX0105     | Men's 100% Cotton Short Sleeve Hero Tee Black       | 62              | 105        | 43               | GREEN: OK for fulfill      | 59                      |
| GGOEGAWH061348   | Onesie Green                                        | 67              | 110        | 43               | GREEN: OK for fulfill      | 59                      |
| GGOEGAAX0334     | Android Men's Long Sleeve Badge Crew Tee Heather    | 44              | 87         | 43               | GREEN: OK for fulfill      | 59                      |
| GGOEGAAR010714   | Men's 100% Cotton Short Sleeve Hero Tee Red         | 48              | 92         | 44               | GREEN: OK for fulfill      | 60                      |
| GGOEGAAJ032817   | Men's Long & Lean Tee Charcoal                      | 58              | 102        | 44               | GREEN: OK for fulfill      | 60                      |
| GGOEGCBB074399   | Device Holder Sticky Pad                            | 49              | 94         | 45               | GREEN: OK for fulfill      | 61                      |
| GGOEGAAX0342     | Women's Vintage Hero Tee White                      | 127             | 172        | 45               | GREEN: OK for fulfill      | 61                      |
| GGOEAAEJ033414   | Android Men's Long Sleeve Badge Crew Tee Heather    | 49              | 95         | 46               | GREEN: OK for fulfill      | 62                      |
| GGOEWFKA083299   | Mood Ninja Window Decal                             | 48              | 94         | 46               | GREEN: OK for fulfill      | 62                      |
| GGOEGAFB035815   | Men's  Zip Hoodie                                   | 52              | 98         | 46               | GREEN: OK for fulfill      | 62                      |
| GGOEGALJ034413   | Women's Vintage Hero Tee Platinum                   | 48              | 94         | 46               | GREEN: OK for fulfill      | 62                      |
| GGOEAAAQ032013   | Android Men's Short Sleeve Hero Tee White           | 49              | 95         | 46               | GREEN: OK for fulfill      | 62                      |
| GGOEGCMB020932   | Suitcase Organizer Cubes                            | 295             | 342        | 47               | GREEN: OK for fulfill      | 63                      |
| GGOEGAEJ028013   | Women's Short Sleeve Hero Tee Grey                  | 70              | 117        | 47               | GREEN: OK for fulfill      | 63                      |
| GGOEGAAX0731     | Men's Fleece Hoodie Black                           | 94              | 142        | 48               | GREEN: OK for fulfill      | 64                      |
| GGOEGADJ056815   | Men's Watershed Full Zip Hoodie Grey                | 57              | 105        | 48               | GREEN: OK for fulfill      | 64                      |
| GGOEGOCR078499   | Spiral Leather Journal                              | 401             | 449        | 48               | GREEN: OK for fulfill      | 64                      |
| GGOEGAAX0795     | 25L Classic Rucksack                                | 66              | 114        | 48               | GREEN: OK for fulfill      | 64                      |
| GGOEGAAX0363     | Women's Long Sleeve Tee Lavender                    | 80              | 128        | 48               | GREEN: OK for fulfill      | 64                      |
| GGOEGAAJ057015   | Men's Short Sleeve Performance Badge Tee Charcoal   | 88              | 137        | 49               | GREEN: OK for fulfill      | 65                      |
| GGOEGEVA022399   | Micro Wireless Earbud                               | 185             | 234        | 49               | GREEN: OK for fulfill      | 65                      |
| GGOEGAAJ032315   | Men's Short Sleeve Hero Tee Charcoal                | 133             | 182        | 49               | GREEN: OK for fulfill      | 65                      |
| GGOEGAEB027814   | Women's Short Sleeve Hero Tee Black                 | 124             | 173        | 49               | GREEN: OK for fulfill      | 65                      |
| GGOEGAAJ080615   | Men's Bike Short Sleeve Tee Charcoal                | 78              | 129        | 51               | GREEN: OK for fulfill      | 66                      |
| GGOEGKWQ060910   | Bib White                                           | 65              | 116        | 51               | GREEN: OK for fulfill      | 66                      |
| GGOEGAAX0325     | Men's Short Sleeve Hero Tee Charcoal                | 53              | 104        | 51               | GREEN: OK for fulfill      | 66                      |
| GGOEGBJL013999   | Canvas Tote Natural/Navy                            | 551             | 602        | 51               | GREEN: OK for fulfill      | 66                      |
| GGOEGAAJ059115   | Men's Short Sleeve Performance Badge Tee Pewter     | 77              | 131        | 54               | GREEN: OK for fulfill      | 67                      |
| GGOEGAAX0351     | Men's Vintage Henley                                | 79              | 133        | 54               | GREEN: OK for fulfill      | 67                      |
| GGOEGETB023799   | Power Bank                                          | 156             | 210        | 54               | GREEN: OK for fulfill      | 67                      |
| GGOEGAAH034016   | Men's Vintage Badge Tee Sage                        | 193             | 248        | 55               | GREEN: OK for fulfill      | 68                      |
| GGOEACCQ017299   | Android Lunch Kit                                   | 107             | 163        | 56               | GREEN: OK for fulfill      | 69                      |
| GGOEGAAX0356     | Men's Vintage Tank                                  | 63              | 119        | 56               | GREEN: OK for fulfill      | 69                      |
| GGOEGAAX0358     | Men's  Zip Hoodie                                   | 178             | 235        | 57               | GREEN: OK for fulfill      | 70                      |
| GGOEAAAJ031914   | Android Men's Short Sleeve Hero Tee Heather         | 69              | 126        | 57               | GREEN: OK for fulfill      | 70                      |
| GGOEGAAB010516   | Men's 100% Cotton Short Sleeve Hero Tee Black       | 90              | 148        | 58               | GREEN: OK for fulfill      | 71                      |
| GGOEGAWN062749   | Infant Zip Hood Pink                                | 89              | 147        | 58               | GREEN: OK for fulfill      | 71                      |
| GGOEGAAX0655     | Toddler Short Sleeve T-shirt Grey                   | 65              | 123        | 58               | GREEN: OK for fulfill      | 71                      |
| GGOEGAAB010515   | Men's 100% Cotton Short Sleeve Hero Tee Black       | 134             | 193        | 59               | GREEN: OK for fulfill      | 72                      |
| GGOEGAEC033115   | Men's Long Sleeve Raglan Ocean Blue                 | 66              | 125        | 59               | GREEN: OK for fulfill      | 72                      |
| GGOEWEBB082699   | Mobile Phone Vent Mount                             | 68              | 129        | 61               | GREEN: OK for fulfill      | 73                      |
| GGOEAAAJ032914   | Android Men's Long & Lean Badge Tee Charcoal        | 68              | 129        | 61               | GREEN: OK for fulfill      | 73                      |
| GGOEGAAX0781     | Leather Journal                                     | 119             | 181        | 62               | GREEN: OK for fulfill      | 74                      |
| GGOEGAAB033813   | Men's Vintage Badge Tee Black                       | 72              | 135        | 63               | GREEN: OK for fulfill      | 75                      |
| GGOEGBJC014399   | Tote Bag                                            | 92              | 155        | 63               | GREEN: OK for fulfill      | 75                      |
| GGOEGAFB035814   | Men's  Zip Hoodie                                   | 90              | 156        | 66               | GREEN: OK for fulfill      | 76                      |
| GGOEGAEJ028015   | Women's Short Sleeve Hero Tee Grey                  | 72              | 138        | 66               | GREEN: OK for fulfill      | 76                      |
| GGOEYAFB073115   | Men's Fleece Hoodie Black                           | 108             | 175        | 67               | GREEN: OK for fulfill      | 77                      |
| GGOEGAAX0629     | Baby Essentials Set                                 | 126             | 194        | 68               | GREEN: OK for fulfill      | 78                      |
| GGOEGHPB080410   | 5-Panel Snapback Cap                                | 151             | 219        | 68               | GREEN: OK for fulfill      | 78                      |
| GGOEYAFB073114   | Men's Fleece Hoodie Black                           | 74              | 143        | 69               | GREEN: OK for fulfill      | 79                      |
| GGOEAHPJ074410   | Android Twill Cap                                   | 91              | 161        | 70               | GREEN: OK for fulfill      | 80                      |
| GGOEGAAQ033916   | Men's Vintage Badge Tee White                       | 85              | 156        | 71               | GREEN: OK for fulfill      | 81                      |
| GGOEGAAX0339     | Men's Vintage Badge Tee White                       | 137             | 210        | 73               | GREEN: OK for fulfill      | 82                      |
| GGOEADHH073999   | Android 17oz Stainless Steel Sport Bottle           | 210             | 283        | 73               | GREEN: OK for fulfill      | 82                      |
| GGOEGOAB022499   | Satin Black Ballpoint Pen                           | 403             | 477        | 74               | GREEN: OK for fulfill      | 83                      |
| GGOEGOBC078699   | Luggage Tag                                         | 629             | 713        | 84               | GREEN: OK for fulfill      | 84                      |
| GGOEGAAH034014   | Men's Vintage Badge Tee Sage                        | 193             | 277        | 84               | GREEN: OK for fulfill      | 84                      |
| GGOEAFKQ020599   | Android Sticker Sheet Ultra Removable               | 363             | 449        | 86               | GREEN: OK for fulfill      | 85                      |
| GGOEGALB034115   | Women's Vintage Hero Tee Black                      | 124             | 213        | 89               | GREEN: OK for fulfill      | 86                      |
| GGOEGAAJ032815   | Men's Long & Lean Tee Charcoal                      | 161             | 251        | 90               | GREEN: OK for fulfill      | 87                      |
| GGOEGPJC203399   | Crunch Noise Dog Toy                                | 262             | 352        | 90               | GREEN: OK for fulfill      | 87                      |
| GGOEGAAQ010414   | Men's 100% Cotton Short Sleeve Hero Tee White       | 94              | 186        | 92               | GREEN: OK for fulfill      | 88                      |
| GGOEAAAJ031915   | Android Men's Short Sleeve Hero Tee Heather         | 101             | 193        | 92               | GREEN: OK for fulfill      | 88                      |
| GGOEGOBG023599   | Colored Pencil Set                                  | 269             | 367        | 98               | GREEN: OK for fulfill      | 89                      |
| GGADFBSBKS42347  | PC gaming speakers                                  | 0               | 100        | 100              | GREEN: OK for fulfill      | 90                      |
| GGOEGESB015099   | Basecamp Explorer Powerbank Flashlight              | 200             | 306        | 106              | GREEN: OK for fulfill      | 91                      |
| GGOEGBRJ037299   | Alpine Style Backpack                               | 165             | 272        | 107              | GREEN: OK for fulfill      | 92                      |
| GGOEGAAQ010415   | Men's 100% Cotton Short Sleeve Hero Tee White       | 221             | 335        | 114              | GREEN: OK for fulfill      | 93                      |
| GGOEGAWQ062953   | Baby Essentials Set                                 | 231             | 346        | 115              | GREEN: OK for fulfill      | 94                      |
| GGOEGHPJ080310   | Blackout Cap                                        | 299             | 415        | 116              | GREEN: OK for fulfill      | 95                      |
| GGOEGAAQ033914   | Men's Vintage Badge Tee White                       | 121             | 237        | 116              | GREEN: OK for fulfill      | 95                      |
| GGOENEBQ079199   | Protect Smoke + CO White Wired Alarm-USA            | 1244            | 1361       | 117              | GREEN: OK for fulfill      | 96                      |
| GGOEGESC014099   | Rocket Flashlight                                   | 123             | 242        | 119              | GREEN: OK for fulfill      | 97                      |
| GGOEGDHR082199   | 25 oz Red Stainless Steel Bottle                    | 221             | 341        | 120              | GREEN: OK for fulfill      | 98                      |
| GGOEGBMC056599   | Waterproof Gear Bag                                 | 140             | 261        | 121              | GREEN: OK for fulfill      | 99                      |
| GGOEGAEB027813   | Women's Short Sleeve Hero Tee Black                 | 130             | 252        | 122              | GREEN: OK for fulfill      | 100                     |
| GGOEGAAX0318     | Men's Short Sleeve Hero Tee Black                   | 151             | 275        | 124              | GREEN: OK for fulfill      | 101                     |
| GGOEYOLR018699   | Leatherette Notebook Combo                          | 1148            | 1276       | 128              | GREEN: OK for fulfill      | 102                     |
| GGOEGBCR024399   | Lunch Bag                                           | 160             | 290        | 130              | GREEN: OK for fulfill      | 103                     |
| GGOEGOCB078299   | Leather Journal-Black                               | 224             | 354        | 130              | GREEN: OK for fulfill      | 103                     |
| GGOEGAAJ032814   | Men's Long & Lean Tee Charcoal                      | 163             | 295        | 132              | GREEN: OK for fulfill      | 104                     |
| GGOEAHPA004110   | Android Wool Heather Cap Heather/Black              | 168             | 305        | 137              | GREEN: OK for fulfill      | 105                     |
| GGOEGAWQ062949   | Baby Essentials Set                                 | 211             | 349        | 138              | GREEN: OK for fulfill      | 106                     |
| GGOEGAAB033815   | Men's Vintage Badge Tee Black                       | 439             | 578        | 139              | GREEN: OK for fulfill      | 107                     |
| GGOEADHH055999   | 22 oz Android Bottle                                | 597             | 738        | 141              | GREEN: OK for fulfill      | 108                     |
| GGOEGCNB021099   | Seat Pack Organizer                                 | 160             | 304        | 144              | GREEN: OK for fulfill      | 109                     |
| GGOEGAAJ032816   | Men's Long & Lean Tee Charcoal                      | 165             | 309        | 144              | GREEN: OK for fulfill      | 109                     |
| GGOEGAAX0341     | Women's Vintage Hero Tee Black                      | 188             | 332        | 144              | GREEN: OK for fulfill      | 109                     |
| GGOEGBMB073799   | Zipper-front Sports Bag                             | 162             | 307        | 145              | GREEN: OK for fulfill      | 110                     |
| GGOEGHGR019499   | Sunglasses                                          | 1046            | 1195       | 149              | GREEN: OK for fulfill      | 111                     |
| GGOEGDHQ014899   | 20 oz Stainless Steel Insulated Tumbler             | 499             | 652        | 153              | GREEN: OK for fulfill      | 112                     |
| GGOEGPJR018999   | 7 Dog Frisbee                                       | 168             | 324        | 156              | GREEN: OK for fulfill      | 113                     |
| GGOEGAAH034015   | Men's Vintage Badge Tee Sage                        | 226             | 390        | 164              | GREEN: OK for fulfill      | 114                     |
| GGOEWFKA082999   | Baby on Board Window Decal                          | 251             | 415        | 164              | GREEN: OK for fulfill      | 114                     |
| GGOEGAAB033814   | Men's Vintage Badge Tee Black                       | 222             | 395        | 173              | GREEN: OK for fulfill      | 115                     |
| GGOEGOAR013599   | Ballpoint Stylus Pen                                | 769             | 942        | 173              | GREEN: OK for fulfill      | 115                     |
| GGOEGOAC021799   | Ballpoint Pen Blue                                  | 226             | 400        | 174              | GREEN: OK for fulfill      | 116                     |
| GGOEGOCD078199   | Leather Journal-Brown                               | 275             | 451        | 176              | GREEN: OK for fulfill      | 117                     |
| GGOEGDHJ082599   | 17 oz Double Wall Stainless Steel Insulated Bottle  | 456             | 633        | 177              | GREEN: OK for fulfill      | 118                     |
| GGOEGAAX0098     | 7 Dog Frisbee                                       | 197             | 376        | 179              | GREEN: OK for fulfill      | 119                     |
| GGOEGAAX0340     | Men's Vintage Badge Tee Sage                        | 254             | 442        | 188              | GREEN: OK for fulfill      | 120                     |
| GGOEGAAX0081     | Recycled Paper Journal Set                          | 545             | 740        | 195              | GREEN: OK for fulfill      | 121                     |
| GGOEGCVB072399   | Executive Umbrella                                  | 242             | 458        | 216              | GREEN: OK for fulfill      | 122                     |
| GGOEGAAX0104     | Men's 100% Cotton Short Sleeve Hero Tee White       | 528             | 749        | 221              | GREEN: OK for fulfill      | 123                     |
| GGOEGBFC018799   | Electronics Accessory Pouch                         | 330             | 568        | 238              | GREEN: OK for fulfill      | 124                     |
| GGOENEBD084799   | Learning Thermostat 3rd Gen-USA - Copper            | 324             | 564        | 240              | GREEN: OK for fulfill      | 125                     |
| GGOEGBPB021199   | Slim Utility Travel Bag                             | 276             | 516        | 240              | GREEN: OK for fulfill      | 125                     |
| GGOEGAAX0338     | Men's Vintage Badge Tee Black                       | 433             | 677        | 244              | GREEN: OK for fulfill      | 126                     |
| GGOEGOXQ016399   | Badge Holder                                        | 916             | 1186       | 270              | GREEN: OK for fulfill      | 127                     |
| GGOEGOAA017199   | Rubber Grip Ballpoint Pen 4 Pack                    | 1515            | 1791       | 276              | GREEN: OK for fulfill      | 128                     |
| GGOEAFKQ020499   | 8 pc Android Sticker Sheet                          | 336             | 613        | 277              | GREEN: OK for fulfill      | 129                     |
| GGOENEBQ084699   | Learning Thermostat 3rd Gen-USA - White             | 849             | 1142       | 293              | GREEN: OK for fulfill      | 130                     |
| GGOEGCKQ013199   | 1 oz Hand Sanitizer                                 | 381             | 679        | 298              | GREEN: OK for fulfill      | 131                     |
| GGOEAKDH019899   | Windup Android                                      | 1351            | 1674       | 323              | GREEN: OK for fulfill      | 132                     |
| GGOEGDWR015799   | Red Shine 15 oz Mug                                 | 1058            | 1417       | 359              | GREEN: OK for fulfill      | 133                     |
| GGOEGOAB021499   | Metal Texture Roller Pen                            | 390             | 775        | 385              | GREEN: OK for fulfill      | 134                     |
| GGOEGCBQ016499   | SPF-15 Slim & Slender Lip Balm                      | 3682            | 4069       | 387              | GREEN: OK for fulfill      | 135                     |
| GGOEGAAB010514   | Men's 100% Cotton Short Sleeve Hero Tee Black       | 681             | 1101       | 420              | GREEN: OK for fulfill      | 136                     |
| GGOEGOAQ020099   | Four Color Retractable Pen                          | 2002            | 2450       | 448              | GREEN: OK for fulfill      | 137                     |
| GGOENEBB078899   | Cam Indoor Security Camera - USA                    | 2139            | 2615       | 476              | GREEN: OK for fulfill      | 138                     |
| GGOEYDHJ056099   | 22 oz  Bottle Infuser                               | 1465            | 1944       | 479              | GREEN: OK for fulfill      | 139                     |
| GGOEGOFH020299   | Screen Cleaning Cloth                               | 844             | 1324       | 480              | GREEN: OK for fulfill      | 140                     |
| GGOEGFKQ020399   | Laptop and Cell Phone Stickers                      | 1033            | 1516       | 483              | GREEN: OK for fulfill      | 141                     |
| GGOEGDHR018499   | 22 oz Water Bottle                                  | 975             | 1482       | 507              | GREEN: OK for fulfill      | 142                     |
| GGOEGHGH019699   | Sunglasses                                          | 1573            | 2086       | 513              | GREEN: OK for fulfill      | 143                     |
| GGOEYHPB072210   | Twill Cap                                           | 1429            | 1997       | 568              | GREEN: OK for fulfill      | 144                     |
| GGOEGBMJ013399   | Sport Bag                                           | 2299            | 2877       | 578              | GREEN: OK for fulfill      | 145                     |
| GGOEGOAJ021599   | Gunmetal Roller Ball Pen                            | 740             | 1321       | 581              | GREEN: OK for fulfill      | 146                     |
| GGOEAOCH077899   | Android Spiral Journal with Pen                     | 665             | 1280       | 615              | GREEN: OK for fulfill      | 147                     |
| GGOEGOCD078399   | Leather Perforated Journal                          | 687             | 1305       | 618              | GREEN: OK for fulfill      | 148                     |
| GGOENEBJ079499   | Learning Thermostat 3rd Gen-USA - Stainless Steel   | 1886            | 2525       | 639              | GREEN: OK for fulfill      | 149                     |
| GGOEGDHC074099   | 17oz Stainless Steel Sport Bottle                   | 716             | 1390       | 674              | GREEN: OK for fulfill      | 150                     |
| GGOEGOAQ012899   | Ballpoint LED Light Pen                             | 1415            | 2098       | 683              | GREEN: OK for fulfill      | 151                     |
| GGOEYOCR077799   | Hard Cover Journal                                  | 1330            | 2046       | 716              | GREEN: OK for fulfill      | 152                     |
| GGOEGEHQ024099   | Clip-on Compact Charger                             | 745             | 1468       | 723              | GREEN: OK for fulfill      | 153                     |
| GGOEGODR017799   | Recycled Mouse Pad                                  | 1033            | 1756       | 723              | GREEN: OK for fulfill      | 153                     |
| GGOEYFKQ020699   | Custom Decals                                       | 3786            | 4536       | 750              | GREEN: OK for fulfill      | 154                     |
| GGOEGOCC077999   | Spiral Journal with Pen                             | 3896            | 4668       | 772              | GREEN: OK for fulfill      | 155                     |
| GGOEGFKQ020799   | Doodle Decal                                        | 1290            | 2080       | 790              | GREEN: OK for fulfill      | 156                     |
| GGOEGHPB071610   | Twill Cap                                           | 917             | 1755       | 838              | GREEN: OK for fulfill      | 157                     |
| GGOEGOAQ018099   | Pen Pencil & Highlighter Set                        | 1067            | 1939       | 872              | GREEN: OK for fulfill      | 158                     |
| GGOEGESQ016799   | Plastic Sliding Flashlight                          | 1570            | 2561       | 991              | GREEN: OK for fulfill      | 159                     |
| GGOEGOCC077299   | RFID Journal                                        | 1834            | 2839       | 1005             | GREEN: OK for fulfill      | 160                     |
| GGOEGOCL077699   | Hard Cover Journal                                  | 1113            | 2171       | 1058             | GREEN: OK for fulfill      | 161                     |
| GGOEGOAR013099   | Stylus Pen w/ LED Light                             | 1100            | 2164       | 1064             | GREEN: OK for fulfill      | 162                     |
| GGOEGHGC019799   | Sunglasses                                          | 1361            | 2444       | 1083             | GREEN: OK for fulfill      | 163                     |
| GGOEGOCB017499   | Leatherette Journal                                 | 3071            | 4978       | 1907             | GREEN: OK for fulfill      | 164                     |
| GGOENEBQ078999   | Cam Outdoor Security Camera - USA                   | 2719            | 4683       | 1964             | GREEN: OK for fulfill      | 165                     |
| GGOEGFYQ016599   | Foam Can and Bottle Cooler                          | 2442            | 4495       | 2053             | GREEN: OK for fulfill      | 166                     |
| GGOEGOCR017899   | Recycled Paper Journal Set                          | 2524            | 4975       | 2451             | GREEN: OK for fulfill      | 167                     |
| GGOEGAAX0037     | Sunglasses                                          | 4204            | 6805       | 2601             | GREEN: OK for fulfill      | 168                     |
| GGOEGOLC013299   | Spiral Notebook and Pen Set                         | 3582            | 6708       | 3126             | GREEN: OK for fulfill      | 169                     |
| GGOEGAAX0074     | 22 oz Water Bottle                                  | 8942            | 15607      | 6665             | GREEN: OK for fulfill      | 170                     |
| GGOEGDHC018299   | 22 oz Water Bottle                                  | 10075           | 19678      | 9603             | GREEN: OK for fulfill      | 171                     |


# Question 2:

The purchasing department wants to understand our inventory vs. ordered situation more granularly, so they know how to deploy their limited resources.  They only want to work on products which have a negative surplus, meaning the stocklevel is less than the orderedquantity, and we are not going to be able to fulfill what has been promised to the customers.  The purchasing department wants to understand what the ranking of "worst products we are understocked in" is when we only look at absolute quantity of how much we are understock, and then wants to compare that to the situation when we look at the percentage of that understock compared to how much was ordered.  The purchasing department wants to see both ranked lists so they can decide whether to focus first on those products for whom we have a large quantity of individual units that we won't have enough stock to fulfill, or focus on those products for whom we may not have as many individual units lacking, but for which the percentage of units lacking compared to how many were ordered, is very large.  Product 2 separate ranked lists, one ranked on absolute number of units we are lacking, and the other ranked on percentage gap between demand and quantity in inventory.  Only look at those products for which we have fewer units in stock than have been ordered.

### SQL Queries:
```SQL
-- Build initial result set that includes only out of stock products, first.
WITH out_of_stock_products as(
SELECT	sku,
		name,
		orderedquantity,
		stocklevel,
		(stocklevel-orderedquantity) AS surplus_quantity,
		(((stocklevel::real-orderedquantity::real)/orderedquantity::real) * 100)::real AS percentage_surplus
FROM products_clean p
WHERE orderedquantity > stocklevel
)
,

-- Apply 2 window functions to a apply ranking from 2 perspectives: absolute quantity understock vs. percentage understock relative to how much was ordered
out_of_stock_products_both_rankings as (
SELECT	*,
		RANK() OVER (ORDER BY surplus_quantity ASC) AS quantity_rank,
		RANK() OVER (ORDER BY percentage_surplus ASC) AS percentage_rank
FROM out_of_stock_products
)

-- Scenario 1: Show the product list with both quantity and percentage rankings, but ordered by quantity ranking
SELECT	*
FROM out_of_stock_products_both_rankings
ORDER BY quantity_rank;

-- Scenario 2: Show the product list with both quantity and percentage rankings, but ordered by percentage ranking
SELECT	*
FROM out_of_stock_products_both_rankings
ORDER BY percentage_rank;

```
## Answer:

The ranking of which products are most urgent to work on purchasing changes depending on whether we rank based on unit quantity we need to fulfill, versus the quantity we are lacking relative to how much was ordered (percentage ranking).

1. When ranked based on absolute unit quantity we are lacking, the top 3 products to work on are:
- "GGOEGFSR022099": "Kick Ball"
- "GGOEGOLC014299": "Metallic Notebook Set"
- "GGOEGGOA017399": "Maze Pen"

2. When ranked based on quantity lacking relative to what was ordered (percentage rank), the top 3 products to work on are:
- "GGOEGFSR022099": "Kick Ball"
- "GGOEGBJC019999": "Collapsible Shopping Bag"
- "GGOEGOCT019199": "Red Spiral  Notebook"

**In both cases, it looks like the Kick Ball is the very first priority that purchasing department employees should focus on purchasing.**

One last finding is that it may be worthwhile for the inventory and procurement processes to be re-engineered and improved, given the number (15) of products with significant "negative surplus" situations and the potential impact to customer satisfaction if their orders are not fulfilled promptly.

Complete result set is:
| sku            | name                                       | orderedquantity | stocklevel | surplus_quantity | percentage_surplus | quantity_rank | percentage_rank |
|----------------|--------------------------------------------|-----------------|------------|------------------|--------------------|---------------|-----------------|
| GGOEGFSR022099 | Kick Ball                                  | 15170           | 723        | -14447           | -95.23401          | 1             | 1               |
| GGOEGBJC019999 | Collapsible Shopping Bag                   | 1184            | 117        | -1067            | -90.11824          | 4             | 2               |
| GGOEGOCT019199 | Red Spiral  Notebook                       | 316             | 43         | -273             | -86.3924           | 11            | 3               |
| GGOEGGOA017399 | Maze Pen                                   | 1748            | 324        | -1424            | -81.46453          | 3             | 4               |
| GGOEGOLC014299 | Metallic Notebook Set                      | 2718            | 610        | -2108            | -77.55703          | 2             | 5               |
| GGOEGAWQ062948 | Baby Essentials Set                        | 261             | 77         | -184             | -70.498085         | 13            | 6               |
| GGOEGDHQ015399 | 26 oz Double Wall Insulated Bottle         | 845             | 256        | -589             | -69.70414          | 7             | 7               |
| GGOENEBQ079099 | Protect Smoke + CO White Battery Alarm-USA | 999             | 325        | -674             | -67.46747          | 6             | 8               |
| GGOEGETR014599 | Tube Power Bank                            | 218             | 72         | -146             | -66.97247          | 14            | 9               |
| GGOEGKAA019299 | Switch Tone Color Crayon Pen               | 1163            | 388        | -775             | -66.63801          | 5             | 10              |
| GGOEGDHC015299 | 23 oz Wide Mouth Sport Bottle              | 402             | 142        | -260             | -64.67662          | 12            | 11              |
| GGOEGFKA022299 | Keyboard DOT Sticker                       | 480             | 189        | -291             | -60.625            | 10            | 12              |
| GGOEGHGT019599 | Sunglasses                                 | 887             | 376        | -511             | -57.60992          | 8             | 13              |
| GGOEGHPB003410 | Snapback Hat Black                         | 230             | 105        | -125             | -54.347824         | 15            | 14              |
| GGOEYOCR077399 | RFID Journal                               | 935             | 433        | -502             | -53.68984          | 9             | 15              |

# Question 3: 

The marketing group wants to know what are the most effective marketing channels, both against total number of units sold, and average price of units sold.  They are debating whether to put more marketing money into certain channels because they cause a **lot** of units to be sold, or focus more on channels where we may sell fewer units, but we may sell *higher cost** products on average.  Produce a list from the **analytics** table including only data where something was actually sold (i.e. unitssold should be > 0 and cannot be NULL since no units were sold in this situation), and show the total number of units sold and average price of units sold, per marketing channel grouping.  Show both the best channel for highest number of units sold, and also for highest average price for units sold.

### SQL Queries:
```SQL
-- Scenario #1: Showing best channel for highest number of units sold:
SELECT DISTINCT	channelgrouping,
				SUM(unitssold) OVER (PARTITION BY channelgrouping) AS total_num_units_sold,
				AVG(unit_price) OVER (PARTITION BY channelgrouping) AS avg_price_units_sold
FROM analytics_clean
WHERE unitssold IS NOT NULL OR unitssold > 0
ORDER BY total_num_units_sold DESC

-- Scenario #2:  Showing best channel for highest average price of units sold:
SELECT DISTINCT	channelgrouping,
				SUM(unitssold) OVER (PARTITION BY channelgrouping) AS total_num_units_sold,
				AVG(unit_price) OVER (PARTITION BY channelgrouping) AS avg_price_units_sold
FROM analytics_clean
WHERE unitssold IS NOT NULL OR unitssold > 0
ORDER BY avg_price_units_sold DESC

-- Overall Scenario: Adding RANKs against total_num_units_sold and avg_price_units_sold:
WITH channel_avgnumunits_avgpriceunits AS (
	SELECT DISTINCT	channelgrouping,
					SUM(unitssold) OVER (PARTITION BY channelgrouping) AS total_num_units_sold,
					AVG(unit_price) OVER (PARTITION BY channelgrouping) AS avg_price_units_sold
	FROM analytics_clean
	WHERE unitssold IS NOT NULL OR unitssold > 0
)

SELECT
	channelgrouping,
	total_num_units_sold,
	avg_price_units_sold,
	RANK() OVER (ORDER BY total_num_units_sold DESC) rank_by_num_units,
	RANK() OVER (ORDER BY avg_price_units_sold DESC) rank_by_avg_units
FROM channel_avgnumunits_avgpriceunits
ORDER BY rank_by_num_units ASC

```

## Answer:

Interestingly, "Referral" channel gives us both the highest number of units sold **and** highest average price of units sold.  We should maintain (or perhaps even increase) our marketing efforts to incentize or otherwise encourage referral and word-of-mouth sales on our ecommerce site because we sell the most units at the highest average price, via referral.

The "Direct" channel is the next highest for both number of units sold **and** next highest average price of units sold.

An interesting finding is that the "Affiliates" channel, though lowest in the total number of units sold, has the 3rd highest average price.  It may be worthwhile to invest some additional marketing dollars into developing this channel because they end up causing higher price sales, so if we were able to increase the number of units selling through this channel, this might be an untapped area where we could more easily increase our profit.

Complete result set, with the use of ranking by both total_num_units_sold and avg_price_units_sold ("Overall Scenario" in SQL above), is here:


| channelgrouping | total_num_units_sold | avg_price_units_sold | rank_by_num_units | rank_by_avg_units |
|-----------------|----------------------|----------------------|-------------------|-------------------|
| Referral        | 168372               | 73.53505             | 1                 | 1                 |
| Direct          | 113171               | 40.90311             | 2                 | 2                 |
| Organic Search  | 100049               | 23.3003              | 3                 | 6                 |
| Paid Search     | 10569                | 25.99787             | 4                 | 4                 |
| Social          | 10029                | 22.99695             | 5                 | 7                 |
| Display         | 9768                 | 25.33975             | 6                 | 5                 |
| Affiliates      | 2734                 | 34.27459             | 7                 | 3                 |

# Question 4: 

What are our top 10 best-selling products by number of units sold?  Show the productsku, and the definitive name from the products table (not the v2 name captured from allsessions, because those names need to be vetted for accuracy).

**Solution note**:  I am assuming that only the entries in the analytics table with unitssold > 0 are the visits that generated any sales.  This could be a mistaken assumption, but it seems the most reliable way to obtain actual sales numbers because each of those rows is tied to an actual visit to the ecommerce site, whereas the salesbysku.total_ordered and the products.orderequantity could be data unrelated to ecommerce sales.  My Option 1 solution is the one I think is most accurate, though I also present 2 other options to obtain similar data.  It is interesting to compare to see how the top 10 best-selling products "change" depending on the source of the data.  More interrogation of the data sources would be needed to determine which is the most accurate set of table(s) to use.

### SQL Queries:
```SQL
-- Option 1:  Most reliable (in my opinion) using data from analytics_clean table and INNER JOINING to all other tables:
SELECT DISTINCT
	allc.productsku,
	p.name AS definitive_name_from_product_list,
	SUM(ac.unitssold) OVER (PARTITION BY allc.productsku) AS total_units_sold
FROM analytics_clean ac
JOIN allsessions_clean allc
ON ac.visitid = allc.visitid
JOIN products_clean p
ON allc.productsku = p.sku
WHERE ac.unitssold IS NOT NULL OR ac.unitssold > 0
ORDER BY total_units_sold DESC
LIMIT 10


-- Option 2:  Another view, from the salesbysku table, though it is not clear how these were ordered and how the total_ordered value was aggregated:
SELECT
	s.productsku,
	CASE
		WHEN p.name IS NOT NULL OR p.name != '' THEN p.name
		ELSE 'Unknown'
	END AS definitive_name,
	s.total_ordered
FROM salesbysku s
LEFT JOIN products_clean p
ON s.productsku = p.sku
ORDER BY s.total_ordered DESC
LIMIT 10


-- Option 3:  Another view, from the products table only, though it is not clear how these were ordered and how the orderequantity value was aggregated:
SELECT
	name,
	orderedquantity
FROM products_clean
ORDER BY orderedquantity DESC
LIMIT 10
```

## Answer:
Using Option 1, which I think is the most reliable, the top 10 products best-selling products are as follows:
| productsku     | definitive_name_from_product_list                 | total_units_sold |
|----------------|---------------------------------------------------|------------------|
| GGOEGBRJ037299 | Alpine Style Backpack                             | 227              |
| GGOEGCBQ016499 | SPF-15 Slim & Slender Lip Balm                    | 168              |
| GGOEGAAX0592   | Men's Airflow 1/4 Zip Pullover Black              | 56               |
| GGOEGHPB071610 | Twill Cap                                         | 28               |
| GGOENEBQ079199 | Protect Smoke + CO White Wired Alarm-USA          | 28               |
| GGOENEBQ084699 | Learning Thermostat 3rd Gen-USA - White           | 27               |
| GGOEGAAX0366   | Women's Scoop Neck Tee White                      | 23               |
| GGOENEBJ079499 | Learning Thermostat 3rd Gen-USA - Stainless Steel | 21               |
| 9182859        | Toddler Raglan Shirt Blue Heather/Navy            | 20               |
| GGOENEBB078899 | Cam Indoor Security Camera - USA                  | 18               |


Option 2 result set (from salesbysku, joined to products) is:
| productsku     | definitive_name                           | total_ordered |
|----------------|-------------------------------------------|---------------|
| GGOEGOAQ012899 | Ballpoint LED Light Pen                   | 456           |
| GGOEGDHC074099 | 17oz Stainless Steel Sport Bottle         | 334           |
| GGOEGOCB017499 | Leatherette Journal                       | 319           |
| GGOEGOCC077999 | Spiral Journal with Pen                   | 290           |
| GGOEGFYQ016599 | Foam Can and Bottle Cooler                | 253           |
| GGOEGOCB078299 | Leather Journal-Black                     | 250           |
| GGOEGHPJ080310 | Blackout Cap                              | 189           |
| GGOEADHH073999 | Android 17oz Stainless Steel Sport Bottle | 167           |
| GGOEGAAX0037   | Sunglasses                                | 146           |
| GGOENEBQ078999 | Cam Outdoor Security Camera - USA         | 112           |


Option 3 result set (from products table only) is:
| sku            | name                              | orderedquantity |
|----------------|-----------------------------------|-----------------|
| GGOEGFSR022099 | Kick Ball                         | 15170           |
| GGOEGDHC018299 | 22 oz Water Bottle                | 10075           |
| GGOEGAAX0074   | 22 oz Water Bottle                | 8942            |
| GGOEGAAX0037   | Sunglasses                        | 4204            |
| GGOEGOCC077999 | Spiral Journal with Pen           | 3896            |
| GGOEYFKQ020699 | Custom Decals                     | 3786            |
| GGOEGCBQ016499 | SPF-15 Slim & Slender Lip Balm    | 3682            |
| GGOEGOLC013299 | Spiral Notebook and Pen Set       | 3582            |
| GGOEGOCB017499 | Leatherette Journal               | 3071            |
| GGOENEBQ078999 | Cam Outdoor Security Camera - USA | 2719            |


# Question 5: 

The marketing department wants to know if there is any obvious pattern evident from the analytics data, that shows if certain months or days that have higher than average visits to our ecommerce site.  They have asked for all visit data, not just visits that results in any sales.

### SQL Queries:
```SQL
-- Begin base result set with simple GROUP BY function to obtain count of number of visits per date.
WITH total_num_visits_by_date AS (
	SELECT
		COUNT(*) AS numvisits,
		date
	FROM analytics_clean GROUP BY DATE
)
,

-- Refine the initial result set and add calculation on average number of visits on each row, to facilitate the comparison on number of visits on a given day against total average, in the next result set.
visits_on_day_and_avg_over_all_days AS (
	SELECT
		numvisits AS visits_on_this_day,
		date,
		AVG(numvisits) OVER () AS avg_visits_over_all_days
FROM total_num_visits_by_date
)

-- Create final result set which highlights whether a given date's number of total visits is above or below the average over all days.
SELECT
	*, 
	CASE
	WHEN visits_on_this_day > avg_visits_over_all_days THEN 'ABOVE'
 	ELSE 'BELOW'
END AS above_or_below_avg_num_visits
FROM visits_on_day_and_avg_over_all_days
ORDER BY above_or_below_avg_num_visits ASC, visits_on_this_day DESC, date ASC
```

## Answer:

It doesn't appear there is a very obvious pattern for when there are above average number of visits to the website versus when there are below average visits to the website.  The distribution (to just an 'eyeball' inspection, no statistics involved) of months and dates seem fairly balanced between the "above average number of visits" group vs. the "below average number of visits" group.

It is important to note that this data, though there are over 4M rows, is limited to dates in the year 2017 and over just the months of May, June, and July.  So the data is quite limited to making large generalizations over times of the year (e.g. Christmas, Black Friday, New Year's Day, etc.).

One interesting artifact from the result set is that the top 3 dates where number of visits was above average, are May 17, 18 and 24.  These days are around the May Long Weekend.  Perhaps, at least in 2017 in the spring/summer months, people were very motivated to look for new items to purchase on the May Long Weekend because typically May Long Weekend for a lot of Canadians, signifies the beginning of "summer" or warmer weather, and "camping/outdoor" season.

The complete result set is here:
| visits_on_this_day | date       | avg_visits_over_all_days | above_or_below_avg_num_visits |
|--------------------|------------|--------------------------|-------------------------------|
| 67481              | 2017-05-17 | 46248.62366              | ABOVE                         |
| 66502              | 2017-05-18 | 46248.62366              | ABOVE                         |
| 62492              | 2017-05-24 | 46248.62366              | ABOVE                         |
| 61303              | 2017-07-18 | 46248.62366              | ABOVE                         |
| 59681              | 2017-07-31 | 46248.62366              | ABOVE                         |
| 59474              | 2017-07-17 | 46248.62366              | ABOVE                         |
| 58293              | 2017-07-13 | 46248.62366              | ABOVE                         |
| 58251              | 2017-07-26 | 46248.62366              | ABOVE                         |
| 58191              | 2017-06-29 | 46248.62366              | ABOVE                         |
| 57676              | 2017-07-21 | 46248.62366              | ABOVE                         |
| 57180              | 2017-05-16 | 46248.62366              | ABOVE                         |
| 56989              | 2017-08-01 | 46248.62366              | ABOVE                         |
| 56968              | 2017-05-19 | 46248.62366              | ABOVE                         |
| 56579              | 2017-07-19 | 46248.62366              | ABOVE                         |
| 55970              | 2017-05-22 | 46248.62366              | ABOVE                         |
| 55816              | 2017-07-07 | 46248.62366              | ABOVE                         |
| 55488              | 2017-07-05 | 46248.62366              | ABOVE                         |
| 55340              | 2017-07-10 | 46248.62366              | ABOVE                         |
| 55175              | 2017-05-31 | 46248.62366              | ABOVE                         |
| 55066              | 2017-06-01 | 46248.62366              | ABOVE                         |
| 54995              | 2017-07-25 | 46248.62366              | ABOVE                         |
| 54161              | 2017-06-27 | 46248.62366              | ABOVE                         |
| 54124              | 2017-05-25 | 46248.62366              | ABOVE                         |
| 54040              | 2017-07-12 | 46248.62366              | ABOVE                         |
| 53893              | 2017-07-27 | 46248.62366              | ABOVE                         |
| 53872              | 2017-06-28 | 46248.62366              | ABOVE                         |
| 53717              | 2017-05-01 | 46248.62366              | ABOVE                         |
| 53512              | 2017-07-20 | 46248.62366              | ABOVE                         |
| 51939              | 2017-07-14 | 46248.62366              | ABOVE                         |
| 51888              | 2017-06-12 | 46248.62366              | ABOVE                         |
| 51534              | 2017-05-02 | 46248.62366              | ABOVE                         |
| 51391              | 2017-07-24 | 46248.62366              | ABOVE                         |
| 51347              | 2017-06-05 | 46248.62366              | ABOVE                         |
| 51254              | 2017-06-30 | 46248.62366              | ABOVE                         |
| 50553              | 2017-05-09 | 46248.62366              | ABOVE                         |
| 50217              | 2017-06-06 | 46248.62366              | ABOVE                         |
| 50208              | 2017-05-23 | 46248.62366              | ABOVE                         |
| 50079              | 2017-06-26 | 46248.62366              | ABOVE                         |
| 50037              | 2017-06-07 | 46248.62366              | ABOVE                         |
| 49842              | 2017-06-13 | 46248.62366              | ABOVE                         |
| 49262              | 2017-05-03 | 46248.62366              | ABOVE                         |
| 49195              | 2017-07-11 | 46248.62366              | ABOVE                         |
| 48827              | 2017-06-02 | 46248.62366              | ABOVE                         |
| 48783              | 2017-07-28 | 46248.62366              | ABOVE                         |
| 48750              | 2017-05-15 | 46248.62366              | ABOVE                         |
| 48548              | 2017-05-04 | 46248.62366              | ABOVE                         |
| 48537              | 2017-06-08 | 46248.62366              | ABOVE                         |
| 48217              | 2017-07-06 | 46248.62366              | ABOVE                         |
| 48170              | 2017-06-14 | 46248.62366              | ABOVE                         |
| 46616              | 2017-05-11 | 46248.62366              | ABOVE                         |
| 46552              | 2017-05-30 | 46248.62366              | ABOVE                         |
| 46094              | 2017-05-12 | 46248.62366              | BELOW                         |
| 44990              | 2017-05-08 | 46248.62366              | BELOW                         |
| 44309              | 2017-05-10 | 46248.62366              | BELOW                         |
| 44228              | 2017-05-26 | 46248.62366              | BELOW                         |
| 43842              | 2017-06-09 | 46248.62366              | BELOW                         |
| 42461              | 2017-06-22 | 46248.62366              | BELOW                         |
| 42441              | 2017-06-19 | 46248.62366              | BELOW                         |
| 41992              | 2017-06-20 | 46248.62366              | BELOW                         |
| 41377              | 2017-05-21 | 46248.62366              | BELOW                         |
| 41117              | 2017-06-15 | 46248.62366              | BELOW                         |
| 41033              | 2017-06-21 | 46248.62366              | BELOW                         |
| 40887              | 2017-05-05 | 46248.62366              | BELOW                         |
| 40614              | 2017-07-30 | 46248.62366              | BELOW                         |
| 40590              | 2017-06-16 | 46248.62366              | BELOW                         |
| 39681              | 2017-06-23 | 46248.62366              | BELOW                         |
| 39597              | 2017-05-20 | 46248.62366              | BELOW                         |
| 39156              | 2017-07-03 | 46248.62366              | BELOW                         |
| 38978              | 2017-07-01 | 46248.62366              | BELOW                         |
| 38596              | 2017-07-23 | 46248.62366              | BELOW                         |
| 38517              | 2017-07-16 | 46248.62366              | BELOW                         |
| 38516              | 2017-07-09 | 46248.62366              | BELOW                         |
| 37915              | 2017-07-15 | 46248.62366              | BELOW                         |
| 37204              | 2017-07-22 | 46248.62366              | BELOW                         |
| 36644              | 2017-07-29 | 46248.62366              | BELOW                         |
| 36632              | 2017-06-04 | 46248.62366              | BELOW                         |
| 36307              | 2017-07-08 | 46248.62366              | BELOW                         |
| 35867              | 2017-06-11 | 46248.62366              | BELOW                         |
| 34974              | 2017-07-04 | 46248.62366              | BELOW                         |
| 34946              | 2017-05-29 | 46248.62366              | BELOW                         |
| 34728              | 2017-07-02 | 46248.62366              | BELOW                         |
| 33930              | 2017-06-25 | 46248.62366              | BELOW                         |
| 33380              | 2017-05-14 | 46248.62366              | BELOW                         |
| 33024              | 2017-05-28 | 46248.62366              | BELOW                         |
| 31078              | 2017-06-24 | 46248.62366              | BELOW                         |
| 30877              | 2017-06-10 | 46248.62366              | BELOW                         |
| 30135              | 2017-06-18 | 46248.62366              | BELOW                         |
| 29765              | 2017-05-06 | 46248.62366              | BELOW                         |
| 29602              | 2017-06-03 | 46248.62366              | BELOW                         |
| 28809              | 2017-05-13 | 46248.62366              | BELOW                         |
| 28662              | 2017-05-07 | 46248.62366              | BELOW                         |
| 27478              | 2017-05-27 | 46248.62366              | BELOW                         |
| 26174              | 2017-06-17 | 46248.62366              | BELOW                         |
