desc "Dumps messages from a queue to files, also requeueing them"
task :dump, [:queue] => :amqp_connect do |t, args|

  queue_name = args[:queue]
  folder_name = queue_name.gsub(/[^A-Za-z0-9\._]+/,"-")
  message_locations = "#{File.dirname(__FILE__)}/#{folder_name}"

  unless Dir.exists? message_locations
    Dir.mkdir message_locations
  end

  connection = @connection
  read_channel = connection.create_channel

  p args
  queue = read_channel.queue(queue_name, {durable: true})
  puts "Connected to Queue: #{queue_name}"

  if queue.message_count == 0
    puts "No messages to be processed."
    exit(true)
  end

  count = queue.message_count
  puts "  #{count} messages to process"
  read_channel.prefetch(count)

  count.times do |loop|
    delivery_info, _, payload = queue.pop(:ack => true)
    begin
      File.open("#{message_locations}/#{loop}.xml", "w+") { |f| f.write payload }

      if loop >= count || queue.message_count == 0
        read_channel.nack(delivery_info.delivery_tag, false, true)
        exit(0)
      end
    rescue StandardError => e
      $stderr.puts "Something went wrong"
      raise e
    end
  end

end

task :properties do
  # Find a suitable properties file
  propfile = "dumper.properties"
  unless File.exist?(propfile)
    $stderr.puts "No properties file found."
    exit(false)
  end
  require 'java_properties'
  @properties = JavaProperties::Properties.new(propfile)
end

task :amqp_connect => :properties do
  require 'bunny'
  @connection = Bunny.new(@properties[:mq])
  @connection.start
end

