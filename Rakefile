require 'bundler/setup'

task default: :download

def sizes
  require 'dimensions'

  @sizes ||= Hash.new(0).tap do |hash|
    Dir[File.expand_path(File.join('images', 'original', '*'), __dir__)].each do |image|
      dimensions = Dimensions.dimensions(image)
      if dimensions
        hash[Dimensions.dimensions(image)] += 1
      end
    end
  end
end

desc 'Deletes images'
task :clean do
  FileUtils.rm_rf(File.expand_path('images', __dir__))
end

desc 'Reports image sizes'
task :sizes do
  sizes.sort.each do |size,count|
    puts "%4d  #{size * 'x'}" % count
  end

  puts sizes.values.reduce(:+)
end

desc 'Reports aspect ratios'
task :ratios do
  ratios = Hash.new(0)
  sizes.each do |size,count|
    ratio = Rational(*size)
    ratio = case ratio.round(2)
    when 0.61, 0.62, 0.63 # 0.618
      'golden'
    when 0.66, 0.67, 0.68 # 0.666
      Rational(2, 3)
    when 0.70, 0.71, 0.72 # 0.714
      Rational(5, 7)
    when 0.74, 0.75, 0.76 # 0.75
      Rational(3, 4)
    when 0.79, 0.80, 0.81 # 0.8
      Rational(4, 5)
    when 1.00
      Rational(1, 1)
    when 1.50
      Rational(3, 2)
    else
      ratio
    end
    ratios[ratio] += count
  end

  total = sizes.values.reduce(:+).to_f
  ratios.sort_by(&:last).reject{|_,v| v < 5}.each do |ratio,count|
    puts "%4d  %4.1f%%  %.2f  #{ratio}" % [count, count / total.to_f * 100, ratio == 'golden' ? 0.61803398875 : Float(ratio)]
  end

  puts total.to_i
end

desc 'Creates an HTML page of images'
task :html do
  require 'dimensions'

  File.open(File.expand_path('images.html', __dir__), 'w') do |f|
    f.write %(<!DOCTYPE html>\n<title></title>\n<body style="margin:0">\n)
    Dir[File.expand_path(File.join('images', '60x90', '*'), __dir__)].reject do |image|
      # Exclude images with beveling, borders, uncentered, silhouettes, errors, etc.
      File.basename(image)[/\A(?:www.ville.brossard.qc.ca|www.gov.mb.ca|www.haldimandcounty.on.ca|www.hamilton.ca)_|_(silhouette|jonesyvonne|eddie_francis|bennett-bill|krog-leonard|mcrae-don|wilkinson-andrew)/]
    end.shuffle.each_slice(32) do |images|
      f.write %(<div style="width:1920px">\n)
      images.each do |image|
        f.write %(<img src="#{image}" width="60" style="float:left">\n)
      end
      f.write %(</div>\n)
    end
  end
end

desc 'Resizes images'
task :resize do
  %w(60x90).each do |size|
    FileUtils.mkdir_p(File.expand_path(File.join('images', size), __dir__))
    Dir[File.expand_path(File.join('images', 'original', '*'), __dir__)].each do |image|
      output = `convert #{image} -resize #{size}^ -gravity center -extent #{size} #{File.expand_path(File.join('images', size, File.basename(image)), __dir__)} 2>&1`
      unless output.empty?
        puts "Error converting #{image}"
      end
    end
  end
end

desc 'Downloads images from Represent'
task :download do
  require 'set'
  require 'uri'

  require 'active_support/cache'
  require 'govkit-ca'
  require 'mechanize'
  require 'faraday_middleware'
  require 'faraday_middleware/response_middleware'

  %w(cache_represent cache_images images/original).each do |directory|
    FileUtils.mkdir_p(File.expand_path(directory, __dir__))
  end

  client = Faraday.new do |connection|
    connection.request :url_encoded
    connection.response :caching do
      ActiveSupport::Cache::FileStore.new('cache_represent', expires_in: 604800) # 1 week
    end
    connection.adapter Faraday.default_adapter
  end

  photo_urls = Set.new
  offset = 0
  begin
    api = GovKit::CA::Represent.new(client)
    response = api.representatives(limit: 1000, offset: offset)
    response['objects'].each do |representative|
      unless representative['photo_url'].empty?
        photo_urls << representative['photo_url']
      end
    end
    offset += 1000
  end until response['meta']['next'].nil?

  # @see https://github.com/opencivicdata/scrapers-ca/issues/72
  photo_urls.reject! do |photo_url|
    parsed = URI.parse(URI.escape(photo_url))
    %w(regina.ca www.calgary.ca www.cityofkingston.ca www.strathcona.ca).include?(parsed.host) && parsed.path[%r{(?:/|\.aspx|/district\d+|/gerretsen)\z}]
  end

  client = Faraday.new do |connection|
    connection.request :url_encoded
    connection.use FaradayMiddleware::FollowRedirects
    connection.response :caching do
      ActiveSupport::Cache::FileStore.new('cache_images')
    end
    connection.adapter Faraday.default_adapter
  end

  filenames = {}
  success = true

  photo_urls.each do |photo_url|
    begin
      url = URI.escape(photo_url)
      response = client.get(url)

      if response.status == 200
        parsed = URI.parse(url)
        host = parsed.host

        if response.headers.key?('content-disposition')
          filename = Mechanize::HTTP::ContentDispositionParser.parse(response.headers['content-disposition']).filename
        else
          filename = URI.unescape(parsed.path)[1..-1]
          if host == 'cms.burlington.ca'
            filename = "#{parsed.query[/\d+\z/]}_#{filename}"
          end
        end

        if response.headers.key?('content-type')
          extension = MIME::Types[response.headers['content-type']].first.extensions.last
        else
          extension = File.extname(filename)[1..-1]
        end
        unless %w(gif jpg png).include?(extension)
          raise "Unrecognized extension '#{extension}' for #{photo_url}"
        end

        # The content-type extension may already be the current extension.
        unless extension == File.extname(filename)[1..-1]
          filename = "#{filename}.#{extension}"
        end

        if filenames.key?(filename)
          raise "Already saved a file named '#{filename}' (#{photo_url})"
        else
          filenames[filename] = true
        end

        filename = filename.force_encoding('utf-8').tr(
          "ÀÁÂÃÄÅàáâãäåĀāĂăĄąÇçĆćĈĉĊċČčÐðĎďĐđÈÉÊËèéêëĒēĔĕĖėĘęĚěĜĝĞğĠġĢģĤĥĦħÌÍÎÏìíîïĨĩĪīĬĭĮįİıĴĵĶķĸĹĺĻļĽľĿŀŁłÑñŃńŅņŇňŉŊŋÒÓÔÕÖØòóôõöøŌōŎŏŐőŔŕŖŗŘřŚśŜŝŞşŠšſŢţŤťŦŧÙÚÛÜùúûüŨũŪūŬŭŮůŰűŲųŴŵÝýÿŶŷŸŹźŻżŽž",
          "AAAAAAaaaaaaAaAaAaCcCcCcCcCcDdDdDdEEEEeeeeEeEeEeEeEeGgGgGgGgHhHhIIIIiiiiIiIiIiIiIiJjKkkLlLlLlLlLlNnNnNnNnnNnOOOOOOooooooOoOoOoRrRrRrSsSsSsSssTtTtTtUUUUuuuuUuUuUuUuUuUuWwYyyYyYZzZzZz"
        ).gsub(%r{['()/ ]|%20}, '_').downcase

        filepath = File.expand_path(File.join('images', 'original', "#{host}_#{filename}"), __dir__)
        unless File.exist?(filepath)
          open(filepath, 'wb') do |f|
            f.write(response.body)
          end
        end

        print '.'
        success = true
      else
        puts "#{"\n" if success}#{response.status} #{photo_url}"
        success = false
      end
    rescue FaradayMiddleware::RedirectLimitReached
      puts "#{"\n" if success}RED #{photo_url}"
      success = false
    end
  end
end
