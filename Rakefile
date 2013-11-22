desc "Dump messages from queue back into queue"
task :dump, [:queue] => :amqp_connect do |t, args|

  queue_name = args[:queue]
  message_locations = "#{File.dirname(__FILE__)}/#{queue_name}"

  unless Dir.exists? "#{File.dirname(__FILE__)}/#{queue_name}"
    Dir.mkdir queue_name
  end

  connection = @connection
  read_channel = connection.create_channel
  read_channel.prefetch(1)

  p args
  queue = read_channel.queue(queue_name, {durable: true})
  puts "Connected to Queue: #{queue_name}"

  if queue.message_count == 0
    puts "No messages to be processed."
    exit(true)
  end

  count = queue.message_count
  puts "  #{count} messages to process"

  loop = 1
  count.times do
    delivery_info, _, payload = queue.pop(:ack => true)
    begin
      File.open("#{message_locations}/#{Time.now.to_i}#{delivery_info.delivery_tag}.xml", "w+") { |f| f.write payload }

      if loop == count
        read_channel.nack(delivery_info.delivery_tag, false, true)
        exit(0)
      end
      loop += 1
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

