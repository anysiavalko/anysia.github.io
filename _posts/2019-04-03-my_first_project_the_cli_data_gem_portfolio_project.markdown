---
layout: post
title:      "My First Project: The CLI Data Gem Portfolio Project"
date:       2019-04-03 11:56:52 -0400
permalink:  my_first_project_the_cli_data_gem_portfolio_project
---


I was (hopefully like many other students) pretty intimidated by the CLI project, but my anxiety was mitigated by my interest in the topic I chose: birthstones.  Yeah, birthstones, as in the gemstones that represent different months of birth.  Don't lie, you know you've looked yours up before.  I have a significant and ever-growing interest (and to my husband's dismay, collection) in different types of gemstones, so a part of me couldn't wait to get started making a cute little custom application related to my favorite new hobby.  Yay!

I watched the Daily Deal video, reviewed every scraping lab we had done and all the notes I took during the labs, studied and copied the video on how to use the downloadable version of the IDE  to make a project from scratch, pressed the purple "help" button twice while trying to figure out the basics of the IDE, and then did my best to imitate what the teacher did in the Daily Deal video, while also looking at the 50 Best Restaurants code in Github.  I thought I had done it right, but when I ran my application, the list of months would only return January, and I was also only able to get extra information for January when I tried to go one level deeper.  

I asked to have a help chat with Beth Schofield, and she explained the major things I was doing incorrectly.  Due to my overcommitment to mimicking the Daily Deal code, I was scraping in my Stone class, and leaving my scraper file empty.  I was also not iterating correctly- I was assigning new stones the properties I had scraped, but I wasn't using a larger section of the HTML to iterate over in the process of creating new stones with said properties.  After that, I started scraping data for new stones in my scraper.rb file, and left the Stone class with just its attribute accessors, initialize and all methods, and a class method that contained the scrape method I created in my Scraper class.  That way, only the Stone class interacted with the Scraper class, and the CLI class only interacted with the Stone class--keeping everything neatly connected, instead of all mixed up and confusing.  I also made sure that I created new stones with scraped properties from one larger section of the gem website data that seemed to contain all the gem data I needed.  I just iterated with .each to assign the different gem data within that section to each new stone.  

However, even with all of those significant improvements, I was still getting, strangely, the same error.  I remembered that Beth had suggested I could also reach out to Jenn Hansen, so I sent her a message, and she had some very helpful advice- first, the section I was iterating over was too large, and I should break it down into consecutively smaller sections before pulling from one single section.  I was pulling from the entire document (doc.css), when I should have selected over the document to get the smallest section possible before iterating.  Second, on a related note, I was appending .first to all of my selectors, in an attempt to not get a huge string of 24 months back at a time, but that's also why my program was not returning data on any months other than January.  The theory was once I whittled my section down, I wouldn't have to use .first.  

I knew that following that advice should have worked, but somehow, it wasn't--I mean, I was no longer only getting back January data, but I was getting back the long strings of 24 months at a time.  I tried putting in a while loop to prevent it from taking any more than 12 selections of data, but instead that returned exactly 12 selections of the same list of 24 months :).  I thought, maybe it's the website?  (It's not me, it's you?)  The website was a really nice site with collapsible windows, where when you clicked on a gem, a smaller window would pop up containing all of the data I needed--I thought, maybe it's too hard to scrape a site with all these windows.  I asked Jenn again, and showed her the code I had, and she pointed out that I was using the word "stone" to represent each division of the section |stone| in my iteration, as well as the object Birthstones::Stone.new.  Also, by using "stone**s**" plural, it may have been searching all of the stone sections, instead of just one at a time.

```
def self.scrape_gia
   website = Nokogiri::HTML(open("https://www.gia.edu/birthstones"))
   section = website.css("div.gem-library").css("div.grey")
   stones = section.css("div.row")
   i = 0
   while i < 12
     section.each do |stone|
       stone = Birthstones::Stone.new
       stone.name = stones.css("strong").text.capitalize
       stone.month = stones.css("div.title").text
       stone.overview = stones.css("p.facts").text
       stone.learn_more = stones.css("a").attr("href").value
       i += 1
     end
 end
```


Once I changed the section name to "gemstone" instead of "stones", it worked perfectly!  

```
 def self.scrape_gia
    website = Nokogiri::HTML(open("https://www.gia.edu/birthstones"))
    section = website.css("div.gem-library").css("div.grey")
    gemstone = section.css("div.row")

    section.each do |gemstone|
      stone = Birthstones::Stone.new
      stone.name = gemstone.css("p.description").text
      stone.month = gemstone.css("div.title").text
      stone.overview = gemstone.css("p.facts").text
      stone.learn_more = gemstone.css("a").last.attr("href")
    end
  end
```

Everything was awesome saucesome.  Except.... when I ran the application and the CLI spit out the list of months, there was a really awkward break in between April and May.

```
Birthstones by Month:
1. January - The birthstone for this month is garnet.
2. February - The birthstone for this month is amethyst.
3. March - The birthstones for this month are aquamarine and bloodstone.
4. April - The birthstone for this month is diamond
.
5. May - The birthstone for this month is emerald.
6. June - The Birthstones for this month are pearl, alexandrite, and moonstone.
7. July - The birthstone for this month is ruby.
8. August - The birthstones for this month are peridot, spinel, and sardonyx.
9. September - The birthstone for this month is sapphire.
10. October - The birthstones for this month are opal and tourmaline.
```

Oh no.  I looked back at the HTML on the gem website, and I saw that for some reason, after the word April, there was a line break after the word Diamond, and then an &nbsp;.  

![](https://imgur.com/sSl56iZ)

I tried tons of string methods to remove the extra space after the text for stone.name (chomp, strip, delete, gsub, etc.), but nothing was working.  So I had to do the thing I really didn't want to do-- make an exception in my code specifically for the month of April.  I took just the strong text for the word April, instead of grabbing all the text in the description (something I ended up doing when I realized that if there were more than one stone representing a particular month, the program would spit back a string of both stones put together), and made a special name variable just for April.  Then, in my CLI, I inserted an if-statement so that there was no break in the list returned from the iteration, and so that if the user selected more information for April, a unique return would be created.  

After that, I was very pleased with my application, very happy to be done with it, and ready to move on.


My final scraping method:

```
def self.scrape_gia
    website = Nokogiri::HTML(open("https://www.gia.edu/birthstones"))
    section = website.css("div.gem-library").css("div.grey")
    gemstone = section.css("div.row")

    section.each do |gemstone|
      stone = Birthstones::Stone.new
      stone.name = gemstone.css("p.description").text
      stone.nameforapril = gemstone.css("p strong").text
      stone.month = gemstone.css("div.title").text
      stone.overview = gemstone.css("p.facts").text
      stone.learn_more = gemstone.css("a").last.attr("href")
    end
  end
```

My CLI code:

```
class Birthstones::CLI

  def call
    call_birthstones
    main_menu
  end

  def call_birthstones
    puts "Birthstones by Month:"
    Birthstones::Stone.get_stones
    Birthstones::Stone.all.each_with_index do |stone, i|
      if i == 3
        puts "#{i + 1}. #{stone.month} - The birthstone for this month is #{stone.nameforapril}."
      else
        puts "#{i + 1}. #{stone.month} - #{stone.name}."
      end
    end
    puts "Every month has its own special birthstone. Above is a list of each month's birthstone."
  end

  def list_birthstones
    puts "Birthstones by Month:"
    Birthstones::Stone.all.each_with_index do |stone, i|
      if i == 3
        puts "#{i + 1}. #{stone.month} - The birthstone for this month is #{stone.nameforapril}."
      else
        puts "#{i + 1}. #{stone.month} - #{stone.name}."
      end
    end
  end

  def main_menu
    input = nil
    while input != "done"
      puts "If you would like to find out more about a particular birthstone, please enter just the number of the month. For example, to view July's birthstone, type '7'."
      puts "You can always type 'list' to view the list of birthstones by month again, or 'done' to exit the program."
      input = gets.strip.downcase
      if input.to_i > 0
        stone = Birthstones::Stone.all[input.to_i-1]
        if input.to_i-1 == 3
          puts "#{stone.month} - The birthstone for this month is #{stone.nameforapril}."
        else
          puts "#{stone.month} - #{stone.name}."
        end
        puts "Overview:\n #{stone.overview}"
        puts "To learn more, visit #{stone.learn_more}"
      elsif input == "list"
        list_birthstones
      elsif input == "done"
        done
      else
        puts "Not a valid input. Please type the number of a month, 'list', or 'done'."
      end
    end
  end

  def done
    puts "Thank you for using my program!"
  end

end
```











