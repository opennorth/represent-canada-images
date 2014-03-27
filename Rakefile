require 'bundler/setup'

require 'set'
require 'uri'

require 'active_support/cache'
require 'govkit-ca'
require 'mechanize'
require 'faraday_middleware'
require 'faraday_middleware/response_middleware'

task default: :download

desc 'Deletes photos'
task :clean do
  FileUtils.rm_rf(File.expand_path('photos', __dir__))
end

desc 'Downloads photos from Represent'
task :download do
  %w(cache_represent cache_photos photos).each do |directory|
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
      ActiveSupport::Cache::FileStore.new('cache_photos')
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
          filename = parsed.path.gsub('/', '_')[1..-1]
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

        filepath = File.expand_path(File.join('photos', "#{host}_#{filename.gsub(/ |%20/, '_').downcase}"), __dir__)
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
