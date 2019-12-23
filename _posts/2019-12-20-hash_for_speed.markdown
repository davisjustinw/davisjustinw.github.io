---
layout: post
title:      "Hash For Speed"
date:       2019-12-20 02:16:01 -0500
permalink:  hash_for_speed
---


My Flatiron Rails portfolio project [DropKeet](https://github.com/davisjustinw/dropkeet) is my attempt at an inventory management tool.  The inspiration for my project came from my past in EMS and remote medicine.  In both fields, tracking medical equipment and supplies can be tricky with subtle challenges that require flexibility.  Spreadsheets quickly become cumbersome and tools on the market were never quite what we needed. 

For a while now, I've had ideas about how I'd want to build a tool.  While some features I envision will probably require Javascript, I felt the Rails project would give me an opportunity to play with some data model ideas and try out an MVP.  So I took a crack at it.

One challenge I faced was with the following view.

![items](https://live.staticflickr.com/65535/49246105781_eeab66a520_o_d.jpg)

The user story this attempts to tackle was that of a stock manager setting par levels across the locations in their charge.  When setting par levels it's easier to be able to see all, if not most of the locations at once, to have something to compare.

The abbreviated columns represent each inventory, while rows represent tracked items in the catalog.  The intersections represent par levels.  The models in this example include the following:

```
class Item
   #name: string
	 
   has_many :inventory_items
   has_many :inventories, through: :inventory_items
end

class Inventory
   #name: string
	 
   has_many :inventory_items
   has_many :items, through: :inventory_items
end

class InventoryItem
   #par :integer
	 
   belongs_to :inventory
   belongs_to :item
end
```

The challenge I faced was that each cell in the table represents an inventory_item but an object may not exist yet.  The cells without a par contain links to a new_inventory_item form.  I essentially had two types of cells that needed to know various combinations of objects to display or route accordingly.  From my perspective I had three potential strategies.

1.  For every inventory or item created, create an inventory_item for each complement (Item -> inventory_item for every item)
2.  Load all items, inventories and inventory_items.  Draw the table using a helper to check if an inventory_item exists for a given inventory/item pair.  If it does, draw the par.  If not, draw the link with parameters.
3.  Load a hash for looking up pars and checking for existence while drawing each cell.

I ruled out 1 because it didn't feel right.  Something in my gut felt like it would be messy to manage creation and destruction of several objects.  I'd probably need to use [Active Record callbacks](https://guides.rubyonrails.org/active_record_callbacks.html).  I felt like there was opportunity for unintended consequences.

I tried 2.  I got it to work, but I couldn't figure out a way to keep from hitting the database several times for each cell.  Ugh.  My sample of 265 items and 6 inventories took several seconds to load.  

Unacceptable user experience.  

I tried various scopes using includes, but just couldn't look up an inventory_item's existence without hitting the database.  Maybe there is a way to do this, I couldn't find the right combination.  I'd love to know if someone has any thougnts.

Ultimately 3 worked the charm.  I built a hash that in combination with a helper worked much faster.  This solution turned out much cleaner codewise than I thought it would be.  So I'm pretty happy.

```
class InventoryItem

...

   # preload items with InventoryItems
   scope :all_loaded, -> { includes(:item) }

   def self.index_hash
      all_loaded.inject({}) do |hash, inv_item|
         item_id = inv_item.item_id
         inv_id = inv_item.inventory_id
         par = inv_item.par
         
				 #build item key if it's not there
         hash[item_id] = {} if !hash[item_id]
				 
				 #add hash with par and the inventory_item object at the proper set of keys
         hash[item_id][inv_id] = { par: par, obj: inv_item }
				 
         hash
      end
   end
end 
```

And here's my InventoryItem helper.

```
module InventoryItemsHelper
   def par_value( par_hash, inventory, item )
	 
	    # Is there an inventory_item? Point to par.
      if par_hash[item.id] && par_hash[item.id][inventory.id]
         link_to edit_inventory_item_path( par_hash[item.id][inventory.id][:obj] ), class: "button" do
           "#{par_hash[item.id][inventory.id][:par]}"
         end
      else
			# Link to new inventory_item.
         link_params = { inventory_item: { inventory_id: inventory.id, item_id: item.id } }
         link_to new_inventory_item_path(link_params), class: "button" do
            "+"
         end
      end
   end
end
```

That's it.  Cheers.

Image hosting provided by [flickr](https://www.flickr.com/)


