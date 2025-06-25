#!/usr/bin/env python3
# etcdctl_collector.py
import os
import subprocess
import logging
from collector import Collector  # Adjust this if you have a collector base class

logger = logging.getLogger('AgensDiagnium.GNULinuxCollector')
logger.setLevel(logging.INFO)

class EtcdCtlInfoCollector(Collector):
    def __init__(self):
        super().__init__('gnulinux', 'gnulinux')
        self.output_dir = "/tools/etcd/cli"
        os.makedirs(self.output_dir, exist_ok=True)

    def run_command_and_save(self, command, output_file):
        try:
            logger.info(f"Running command: {command}")
            result = subprocess.run(
                command, shell=True, text=True, capture_output=True
            )
            with open(output_file, "w") as f:
                f.write(result.stdout)
                if result.stderr:
                    f.write("\n--- STDERR ---\n")
                    f.write(result.stderr)
        except Exception as e:
            logger.error(f"Error while running {command}: {e}", exc_info=True)

    def collect_etcdctl_info(self):
        status_file = os.path.join(self.output_dir, "endpoint_status.out")
        health_file = os.path.join(self.output_dir, "endpoint_health.out")

        self.run_command_and_save("etcdctl endpoint status", status_file)
        self.run_command_and_save("etcdctl endpoint health", health_file)

if __name__ == "__main__":
    collector = EtcdCtlInfoCollector()
    collector.collect_etcdctl_info()
