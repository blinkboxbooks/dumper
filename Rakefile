desc "Dumps messages from a queue to files, also requeueing them"
task :dump, [:queue, :perm_dump, :dlx]  do |t, args|

  get_properties
  connect
  queue_name = args[:queue]
  dlx = args[:dlx]
  perm_dump = ['true', 'ack'].include? args[:perm_dump].to_s.downcase
  folder_name = queue_name.gsub(/[^A-Za-z0-9\._]+/,"-")
  dump_dir = ENV['DUMP_DIR']|| "#{File.dirname(__FILE__)}
  message_locations = "#{dump_dir}/#{folder_name}"

  unless Dir.exists? message_locations
    Dir.mkdir message_locations
  end

  read_channel = @connection.create_channel

  if dlx
    queue = read_channel.queue(queue_name, {durable: true, arguments: {"x-dead-letter-exchange" => "#{dlx}"}})
  else
    queue = read_channel.queue(queue_name, {durable: true})
  end
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
      read_channel.ack(delivery_info.delivery_tag, true) if perm_dump
      if loop >= count || queue.message_count == 0
        # This is is not supported by other AMQP implementations outside of rabbitmq
        # but I find it to be the better of the two options we have.
        #read_channel.nack(delivery_info.delivery_tag, false, true)
        read_channel.close
        exit(0)
      end

    rescue StandardError => e
      $stderr.puts "Could not get messages from the queue. Make sure the provided information in the properties file is correct.\n#{e.message}"
      raise e
    end
  end

end

def get_properties
  # Find a suitable properties file
  propfile = "dumper.properties"
  unless File.exist?(propfile)
    $stderr.puts "No properties file found."
    exit(false)
  end
  require 'java_properties'
  @properties = JavaProperties::Properties.new(propfile)
end

def connect
  require 'bunny'
  @connection = Bunny.new(@properties[:mq])
  @connection.start
end

